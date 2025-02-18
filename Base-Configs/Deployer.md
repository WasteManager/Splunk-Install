# Base Config Files Needed | Location for Install | Config Changes

| Config File                   | Location                      | Config Changes |
|--------------------------------|------------------------------|----------------|
| **org_all_deploymentclient**   | `/opt/splunk/etc/apps`       | `deploymentclient.conf` (Modify: `phoneHomeIntervalInSecs`, `[target-broker:deploymentServer] targetURI = ipandport of deployment server`) |
| **org_all_forwarder_outputs**  | `/opt/splunk/etc/apps`       | `outputs.conf` (Modify: `[tcpout:primary_indexers] server = ipOfIndexer:port, ipOfIndexer2:port, ipOfIndexer3:port`)<br> `[tcpout-server://ipOfIndexer1:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem`<br> `[tcpout-server://ipOfIndexer2:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem`<br> `[tcpout-server://ipOfIndexer3:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem` |
| **org_cluster_manager_base**   | `/opt/splunk/etc/apps`       | `server.conf` (Modify: `[clustering] mode = manager, replication_factor = <value>, search_factor = <value>, pass4SymmKey = <password>, cluster_label = <name>`) |
| **org_full_license_server**    | `/opt/splunk/etc/apps`       | `server.conf` (Modify: `[license] manager_uri: https://ipoflicensemanager:managementport`) |
| **org_all_indexes**            | `/opt/splunk/etc/manager-apps` | `indexes.conf` (Modify: Rename `homePath`, `coldPath`, add `frozenTimePeriodInSecs` to all indexes. Add custom indexes at the bottom. Define cold-to-frozen path: `coldToFrozenDir = /frozen`) |
| **org_cluster_indexer_base**   | `/opt/splunk/etc/manager-apps` | `inputs.conf` (Modify: `[SSL] serverCert = /opt/splunk/etc/auth/mycerts/SplunkServerCert.pem, password = <password>, requireClientCert = False`) <br> `server.conf` (Modify: `[clustering] mode = peer, manager_uri = ipOfClusterManager:8089`)<br> `[replication_port://9100] disabled = false`<br> `[sslConfig] sslRootCertCAPath = /opt/splunk/etc/auth/mycerts/CACertificate.pem`<br> `[license] manager_uri = https://ipOfLicenseManager:managementport` |
| **org_indexer_volume_indexes** | `/opt/splunk/etc/manager-apps` | `indexes.conf` (Modify: `[volume:hot] path = /hot, maxVolumeDataSizeMB = <value>`<br> `[volume:cold] path = /cold, maxVolumeDataSizeMB = <value>`) |
| **Splunk_TA_nix**              | `/opt/splunk/etc/manager-apps` | `defaults` |
| **Splunk_TA_windows**          | `/opt/splunk/etc/manager-apps` | `defaults` |
| **org_search_volume_indexes**  | `/opt/splunk/etc/apps`       | `indexes.conf` (Modify: `[volume:hot] path = /hot`<br> `[volume:cold] path = /cold`) |


