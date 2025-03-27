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

# Situation: Some windows are not sending security logs, and you need to generate a list
  - Ensure that: search date is far enough back, forwarder is correctly installed and configs are correct, ensure the correct app is installed on the machine
  - Initiate the following query
| tstats count where index=windows by host | rename host as all_host | eval has_security=0 | append [ | tstats count where index=windows source=xmlWinEventLog:Security by host | eval has_security=1 | rename host as all_host  | stats max(has_security) as has_security by all_host | has_security=0 | rename all_host as host


