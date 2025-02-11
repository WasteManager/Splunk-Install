# Steps to setting up SSO
  - On the Splunk Search Head UI (or whatever search head you want SSO enabled for users)
  - Settings > AUthentication Methods
  - Chooose SAML then click SAML Settings LInk
  - lick SAML Configurations
  - DOwnload the FederationMEtadata.xml from whereever you have it stored https://xxx.xxxx.hilton.gov/FederationMEtadata/2007-06/FederationMetadata.xml
  - Click and Select File button next to MetadataXML and upload the "FederationMetadata.xml" file from the previous step

  - # General Settings
  - Replicate Certificates = checked
  - Entity ID = https://fqdn.of.your.server
  - # Alias
  - Role Alias = groupROles
  - # Advanced Settings
  - Name Id Format = Transient
  - Fully qualified Domain = https://fqdn.of.your.server
  - Redirect port = 8000
  - SSO Binding = HTTP POst
  - SLO Binding HTTP Post

  - edit this file: /opt/splunk/etc/system/local/authentication.conf
  - [roleMap_SAML]
  - admin = splunk admins;splunk ml users
  - can_delete = splunk can delete
  - db_connect_admin = splunk admins
  - db_connect_user = splunk db connect users
  - exhange-admin = splunk admins
  - power = splunk power users
  - splunk-system-role = splunk admins
  - user = splunk power users;splunk users
  - winfra-admin = splunk admins

  - Run this command /opt/splunk/bin/splunk reload auth

  - *Be sure the following command is ran on the SSO server in an elevated POwerSHell prompt. TargetName should match the rle setup for this Splunk instance
  - Set-AdfsRelyingPartyTrust -TargetName SPLUNKSERVERNAME -SIgningCertficiateREvocationCheck None
