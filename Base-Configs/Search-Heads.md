# Base Config files Needed | Location for install | Config Changes:
- org_all_forwarder_outputs | /opt/splunk/etc/apps | outputs.conf([tcpout:primary_indexers] server = ipOfIndexer:port, ipOfIndexer2:port, ipOIndexer3:port | [tcpout-server://ipOfIndexer1:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem,[tcpout-server://ipOfIndexer2:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem,[tcpout-server://ipOfIndexer3:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem)
- org_all_search_base | /opt/splunk/etc/apps | defaults
- org_full_license_server | /opt/splunk/etc/apps | server.conf([license] manager_uri: https://ipoflicensemanager:managementport)
- Splunk_TA_nix | /opt/splunk/etc/apps | See Technology Add-ons
- Splunk_TA_windows | /opt/splunk/etc/apps | See Technology Add-ons
