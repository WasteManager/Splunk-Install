# Base Config files Needed | Location for install | Config Changes: 

- org_all_deploymentclient | /opt/splunk/etc/apps | deploymentclient.conf(phoneHomeIntervalInSecs, [target-broker:deploymentServer] targetURI = ipandport of deployment server)
- org_all_forwarder_outputs | /opt/splunk/etc/apps | outputs.conf([tcpout:primary_indexers] server = ipOfIndexer:port, ipOfIndexer2:port, ipOIndexer3:port | [tcpout-server://ipOfIndexer1:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem,[tcpout-server://ipOfIndexer2:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem,[tcpout-server://ipOfIndexer3:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem)
- org_cluster_manager_base | /opt/splunk/etc/apps | server.conf([clustering] mode = manager, replication_factor = whateveryouneed, search_factor = whateveryouneed, pass4SymmKey: passwordToBeSharedAmongClusteredIndexers, cluster_label: nameOfYourClusteredIndexerEnvironment)
- org_full_license_server | /opt/splunk/etc/apps | server.conf([license] manager_uri: https://ipoflicensemanager:managementport)
- org_all_indexes | /opt/splunk/etc/manager-apps | indexes.conf(rename all of the homePath, coldPath + add forzenTimePriodInSecs to all indexes, this is where you add all of your custom indexes down at the bottom, define cold to frozen path with: coldToFrozenDir = /frozen)
- org_cluster_indexer_base | /opt/splunk/etc/manager-apps |inputs.conf([SSL]serverCert = /opt/splunk/etc/auth/mycerts/SplunkServerCert.pem, password = passwordOfSplunkServerCert.pem, requireClientCert = False), server.conf([clustering] mode = peer, manager_uri = ipOfClusterManager:8089, [replication_port://9100] disabled = false, [sslConfig] sslRootCertCAPath = /opt/splunk/etc/auth/mycerts/CACertificate.pem, [license] manager_uri = https://ipOfLicenseManager:managementport)
- org_indexer_volume_indexes | /opt/splunk/etc/manager-apps | indexes.conf([volume:hot]path = /hot maxVolumeDataSizeMB = whateverYouNeed(match this to whatever size is allocated to the machine volume), [volume:cold]path = /cold, maxVolumeDataSizeMB = whateverYouNeed(match this to whatever size is allocated to the machine volume))
- Splunk_TA_nix | /opt/splunk/etc/manager-apps | defaults
- Splunk_TA_windows | /opt/splunk/etc/manager-apps | defaults

