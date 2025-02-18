# Base Config files Needed | Location for install | Config Changes: 

- org_all_deploymentclient | /opt/splunk/etc/apps | deploymentclient.conf(phoneHomeIntervalInSecs, [target-broker:deploymentServer] targetURI = ipandport of deployment server)
- org_all_forwarder_outputs | /opt/splunk/etc/apps | outputs.conf([tcpout:primary_indexers] server = ipOfIndexer:port, ipOfIndexer2:port, ipOIndexer3:port | [tcpout-server://ipOfIndexer1:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem,[tcpout-server://ipOfIndexer2:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem,[tcpout-server://ipOfIndexer3:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem)
- org_cluster_manager_base | /opt/splunk/etc/apps | server.conf([clustering] mode = manager, replication_factor = whateveryouneed, search_factor = whateveryouneed, pass4SymmKey: passwordToBeSharedAmongClusteredIndexers, cluster_label: nameOfYourClusteredIndexerEnvironment)
- org_full_license_server | /opt/splunk/etc/apps | server.conf([license] manager_uri: https://ipoflicensemanager:managementport)

