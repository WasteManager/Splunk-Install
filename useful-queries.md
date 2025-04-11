  - Total Events last 24 hours

index=windows_logs sourcetype="WinEventLog" earliest=-24h
| stats count AS Total_Events

  - Event count by severity

index=windows_logs sourcetype="WinEventLog" earliest=-24h
| eval Severity=case(EventCode >= 1 AND EventCode <= 2999, "Information",
                      EventCode >= 3000 AND EventCode <= 3999, "Warning",
                      EventCode >= 4000, "Error",
                      true(), "Unknown")
| stats count BY Severity


  - Event Volume trend over time

index=windows_logs sourcetype="WinEventLog" earliest=-24h
| timechart span=1h count BY EventCode

  - Successful vs. Failed Logins
index=windows_logs sourcetype="WinEventLog:Security" EventCode IN (4624, 4625)
| eval Status=if(EventCode=4624, "Successful Login", "Failed Login")
| timechart span=1h count BY Status

  - Locked out Accounts
index=windows_logs sourcetype="WinEventLog:Security" EventCode=4740
| stats count BY Account_Name, _time

  - Unauthorized access attempts
index=windows_logs sourcetype="WinEventLog:Security" EventCode IN (529, 539, 644, 4740, 4625)
| stats count BY host, Account_Name, EventCode

  - System Reboots
index=windows_logs sourcetype="WinEventLog:System" EventCode IN (6005, 6006)
| eval Event_Type=case(EventCode=6005, "System Start", EventCode=6006, "System Shutdown")
| timechart span=1h count BY Event_Type


  - Service Crashes
index=windows_logs sourcetype="WinEventLog:System" EventCode IN (7031, 7034)
| stats count BY host, Service_Name, EventCode

  - High CPU/Memory loads

index=windows_logs sourcetype="WinEventLog:System" EventCode IN (2019, 2020, 2021)
| stats count BY EventCode, host

  - Login Activity heat map

index=windows_logs sourcetype="WinEventLog:Security" EventCode=4624
| eval hour=strftime(_time, "%H")
| chart count OVER hour BY Account_Name

  - User logins by host

index=windows_logs sourcetype="WinEventLog:Security" EventCode=4624
| stats count BY Account_Name, host

  - Logoff Events
index=windows_logs sourcetype="WinEventLog:Security" EventCode=4647
| stats count BY Account_Name, host

  - Top application logging errors

index=windows_logs sourcetype="WinEventLog:Application" EventCode>=4000
| stats count BY Source
| sort -count

  - Critical Service Start/Stop

index=windows_logs sourcetype="WinEventLog:System" EventCode IN (7036, 7045)
| stats count BY host, Service_Name, EventCode

  - High Freq App crashes
index=windows_logs sourcetype="WinEventLog:Application" EventCode=1000
| stats count BY host, Source
| sort -count

  - Group Policy changes

index=windows_logs sourcetype="WinEventLog:Security" EventCode=4739
| stats count BY Account_Name, Group_Name

  - Priv Escalation

index=windows_logs sourcetype="WinEventLog:Security" EventCode=4672
| stats count BY Account_Name, host

  - Software Installatons

index=windows_logs sourcetype="WinEventLog:Application" EventCode=11707
| stats count BY host, Software_Name

  - Unauthorized access attempts alerts

index=windows_logs sourcetype="WinEventLog:Security" EventCode IN (529, 539, 644, 4740, 4625)
| where _time > relative_time(now(), "-1h")
| stats count BY host, Account_Name

  - New admin User creation

index=windows_logs sourcetype="WinEventLog:Security" EventCode=4720
| stats count BY Account_Name, host

  - Unusual Event Spikes

index=windows_logs sourcetype="WinEventLog"
| bin _time span=1h
| stats count BY _time
| eventstats avg(count) AS avg_count, stdev(count) AS stdev_count
| where count > avg_count + (2 * stdev_count)

- Embedded powershell in command line

index=windows sourcetype=WinEventLog:Security EventCode=4688
| search Command_Line="*powershell*"
| search Command_Line="*-enc*" OR Command_Line="*-e *"
| stats count by Account_Name, Command_Line

 - Executable files embedded /temp
   
index=windows EventCode=4663 Object_Name="*\\Temp\\*.exe"
| stats count by Account_Name, Object_Name, host

 - powershell Command execution
   
index=windows sourcetype=WinEventLog:Security EventCode=4688
| where Process_Name="powershell.exe"
| stats count by Account_Name, Command_Line, host

 - Connection to malicious IP's
   
index=network OR index=firewall
[ | inputlookup threat_intel.csv | fields ip | rename ip as dest_ip ]
| stats count by src_ip, dest_ip, dest_port

 - Most bandwidth used
   
index=netflow OR index=network
| stats sum(bytes_in) as BytesIn, sum(bytes_out) as BytesOut by src_ip
| eval TotalBytes = BytesIn + BytesOut
| sort -TotalBytes

 - Unusual ports accessed
   
index=network OR index=firewall
| stats count by dest_port
| sort -count
| where dest_port > 1024

 - Succesful logins after failures (potential brute force)
   
(index=windows EventCode=4624 OR EventCode=4625)
| stats values(EventCode) as Events by Account_Name, host, Source_Network_Address
| search Events="4624" AND Events="4625"

 - Failed logins by user
   
index=windows EventCode=4625
| stats count by Account_Name, host, Source_Network_Address
| sort -count

 - What Splunk Users are search
   
index=_audit action=search info=granted
| table _time user search info
| sort -_time

 - Splunk Role and Permission changes
   
index=_audit action=edit_user
| table _time user action info
| sort -_time

 - Long or expensive Search Monitoring
   
index=_internal sourcetype=scheduler
| stats avg(run_time) as avg_runtime, max(run_time) as max_runtime by user savedsearch_name
| where avg_runtime > 5000

 - Splunk Login attempts
   
index=_audit action="login attempt"
| stats count by user, info, src, _time
| sort -_time

- DHCP events

index=dhcp | stats couny by descirption, action, signature

 - Total DHCP Events Over Time
index=dhcp sourcetype=dhcpsrvlog
| timechart count by action span=1h

 - Blocked DHCP Requests (per Host)
   
index=dhcp sourcetype=dhcpsrvlog action=blocked
| stats count by host
| sort - count

 - NACK Events with Description

index=dhcp sourcetype=dhcpsrvlog description="NACK"
| table _time host dest_ip mac vendor signature
| sort - _time

 - Top Denied Leases 

index=dhcp sourcetype=dhcpsrvlog signature="A lease was denied"
| stats count by dest_ip mac vendor
| sort - count

 - Vendor Distribution

index=dhcp sourcetype=dhcpsrvlog
| stats count by vendor
| sort - count

 - MAC Address Activity

index=dhcp sourcetype=dhcpsrvlog
| stats count by mac action
| sort - count

 - Daily Summary

index=dhcp sourcetype=dhcpsrvlog
| timechart span=1d count by action

 - Action Breakdown Pie Chart

index=dhcp sourcetype=dhcpsrvlog
| stats count by action

 - Top failed logon attempts by Account (Detect Brute Force/ Spray)

index=windows OR index=MSAD sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Account_Name, src_ip
| sort - count


 - Success Vs Fail logon Trend

index=windows OR index=MSAD sourcetype="WinEventLog:Security" EventCode=4624 OR EventCode=4625
| eval outcome=if(EventCode=4624, "Success", "Failure")
| timechart span=1h count by outcome


 - Top Locked-out Accounts

index=windows OR index=MSAD sourcetype="WinEventLog:Security" EventCode=4740
| stats count by Target_Account_Name
| sort - count

 - Workstation Generating Most Failures

index=windows OR index=MSAD sourcetype="WinEventLog:Security" EventCode=4625
| stats count by Workstation_Name
| sort - count

 - Lateral Movement Detection: Logons from Multiple Hosts

index=windows sourcetype="WinEventLog:Security" EventCode=4624 Logon_Type=10 OR Logon_Type=3
| stats dc(src_ip) AS unique_sources by Target_Account_Name
| where unique_sources > 3

 - High Privilege Account Activity

index=windows OR index=MSAD sourcetype="WinEventLog:Security" EventCode=4624
| lookup domain_admins_lookup Account_Name OUTPUT is_admin
| search is_admin=true
| stats count by Account_Name, src_ip

 - Locked out Accounts Over Time

index=windows OR index=MSAD sourcetype="WinEventLog:Security" EventCode=4740
| timechart span=1h count

 - Auth Attempts Heatmap by Hour

index=windows OR index=MSAD sourcetype="WinEventLog:Security" EventCode=4625
| eval hour=strftime(_time, "%H")
| stats count by hour, Account_Name

 - System Errors and Warnings by Host

index=windows sourcetype="WinEventLog:System" (EventCode=1000 OR Type=Error OR Type=Warning)
| stats count by host, Type
| sort - count

 - Windows Service Crashes / Restarts

index=windows sourcetype="WinEventLog:System" EventCode=7031
| stats count by host, Service_Name
| sort - count

 - System Reboots / Shutdown Tracking

index=windows sourcetype="WinEventLog:System" EventCode=1074
| stats count by host, User, Message
| sort - count

 - Auth Volume Per Host

index=windows sourcetype="WinEventLog:Security" (EventCode=4624 OR EventCode=4625)
| stats count by host, EventCode
| eval AuthType=if(EventCode=4624, "Success", "Failure")
| stats sum(count) as total by host, AuthType

 - Account Lockouts by Host (Operational Noise Indicator)

index=windows sourcetype="WinEventLog:Security" EventCode=4740
| stats count by host
| sort - count

 - Most Verbose Hosts by Event Volume

index=windows
| stats count by host
| sort - count

 - Windows Update Failures

index=windows sourcetype="WinEventLog:System" EventCode=20
| stats count by host
| sort - count

 - Privileged Account Usage (Unexpected Hosts)

index=windows sourcetype="WinEventLog:Security" `eventcode_logon_activity`
| lookup domain_admins_lookup Account_Name OUTPUT Account_Name as is_admin
| search is_admin=*
| stats count by Account_Name, src_ip, host
| sort - count

 - Critical Account Authentication from New Workstations

index=windows sourcetype="WinEventLog:Security" `eventcode_logon_activity`
| lookup critical_accounts_lookup Account_Name OUTPUT Account_Type
| stats dc(Workstation_Name) as unique_workstations by Account_Name
| where unique_workstations > 3
| sort - unique_workstations

 - Excessive Account Lockouts (potential Password spray)

index=windows sourcetype="WinEventLog:Security" `eventcode_account_lockouts`
| stats count by Target_Account_Name, host
| where count > 5
| sort - count


 - Service Crahses (Operational Stability)

index=windows sourcetype="WinEventLog:System" `eventcode_service_failures`
| stats count by host, Service_Name
| sort - count

 - Privileged Group Changes (DA Tampering)

index=windows sourcetype="WinEventLog:Security" (EventCode=4728 OR EventCode=4729 OR EventCode=4732 OR EventCode=4733)
| stats count by Target_Account_Name, Group_Name, host
| sort - count

# Situation: Some windows are not sending security logs, and you need to generate a list
  - Ensure that: search date is far enough back, forwarder is correctly installed and configs are correct, ensure the correct app is installed on the machine
  - Initiate the following query
| tstats count where index=windows by host | rename host as all_host | eval has_security=0 | append [ | tstats count where index=windows source=xmlWinEventLog:Security by host | eval has_security=1 | rename host as all_host  | stats max(has_security) as has_security by all_host | has_security=0 | rename all_host as host


