# Base Config Files Needed | Location for Install | Config Changes

| Config File                   | Location                      | Config Changes |
|--------------------------------|------------------------------|----------------|
| **org_all_deployment_client**  | `/opt/splunk/etc/apps`       | `deploymentclient.conf` (Modify: `phoneHomeIntervalInSecs`, `[target-broker:deploymentServer] targetURI = ipandport of deployment server`) |
| **org_all_forwarder_outputs**  | `/opt/splunk/etc/apps`       | `outputs.conf` (Modify: `[tcpout:primary_indexers] server = ipOfIndexer:port, ipOfIndexer2:port, ipOfIndexer3:port`)<br> `[tcpout-server://ipOfIndexer1:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem`<br> `[tcpout-server://ipOfIndexer2:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem`<br> `[tcpout-server://ipOfIndexer3:port] sslCertPath = /opt/splunk/etc/auth/mycerts/SSLCERT.pem, sslPassword = passwordForSSLCERT.pem` |
| **org_all_uf_outputs**         | `/opt/splunk/etc/apps`       | `outputs.conf` (Modify: `[tcpout] defaultGroup = primary_indexers`, `[tcpout:primary_indexers] server = indexer1:9997, indexer2:9997, indexer3:9997`, `disabled = 0`, `compressed = true`, `sslVerifyServerCert = false`, `sslRootCAPath = /opt/splunk/etc/auth/cacert.pem`, `sslCertPath = /opt/splunk/etc/auth/server.pem`, `sslPassword = password`, `sslVersion = tls1.2`, `autoLBFrequency = 15`)<br> `#forceTimeBasedAutoLB = true` |
| **org_full_license_server**    | `/opt/splunk/etc/apps`       | `server.conf` (Modify: `[license] manager_uri: https://ipoflicensemanager:managementport`) |

