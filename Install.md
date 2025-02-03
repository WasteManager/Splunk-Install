# Misc Install notes
  -  when rpm -i splunk._____rpm
  - (May get an error useradd: cannot create directory /opt/splunk -> check to see if /opt/splunk IS created
  - su - splunk (switch user to splunk user)
  - DO NOT RUN SPLUNK AS ROOT
  - enable splunk to start at reboot (must be set up as root)
  - chown -R splunk:splunk /opt/splunk (if perms do not add up)
  - if PID is not dead then kill -9 #PID
  - 8089 is default management port for splunk
  - ss -plnt # check listening ports
  - deployer only pushes apps down to forwarders
  - cluster manager does the data management
  - ./splunk show shcluster-status (shows if searchhead cluster is up, shows which is captain)
# Best case run
  - turn on machine
  - Use winscp to move splunk rpm to splunk machine
  - chmod 644 splunk-___.rpm
  - chmod +x splunk-___.rpm
  - sudo -s
  - rpm -i splunk-___.rpm
  - su - splunk (turn splunk on and off as splunk user)
  - cd /opt/splunk/bin
  - ./splunk start --accept-license --answer-yes
      -   - Will be prompted to create a local admin account: use admin and easy to remember password(update keepass!!)
          - GO UPDATE THE KEEPASS
  - ./splunk stop
  - exit (go back to root user)
  - cd /opt/splunk/bin
  - ./splunk enable boot-start -systemd-managed 1 -user splunk
  - systemctl start Splunkd
  - # Enable correct ports
  - ensure ports are open (in powershell tnc -p 8000 #target ip address)
  - sudo -s
  - firewall-cmd --zone=public --add-port=8000/tcp --permanent
  - firewall-cmd --zone=public --add-port=8089/tcp --permanent (or use ^8000^8089)
  - firewall-cmd --zone=public --add-port=9997/tcp --permanent (or use ^8089^9997)
  - firewall-cmd --zone=public --add-port=9100/tcp --permanent #9100 being used for replication port for indexers
  - firewall-cmd --zone=public --add-port=9200/tcp --permanent #Replication across search heads only)
  - firewall-cmd --zone=public --add-port=8191/tcp --permanent (critical for kvstore[where clusterd knowledged objects to replicated too])
  - firewall-cmd --reload #rereads to make what a change, (puts things in effect)

  # Set up Cluster Manager
  - Log into cluster master machine backend
  - Move base config file (this is tribal knowledge) to /tmp (using CDS)
  - move into /tmp
  - change ownership: chown -R splunk:splunk base_config_file_manager_base
  - change name if desired to something unique ie: J_base_config_file_manager_base
  - cd to /opt/splunk/etc/apps/j_base_config_file_manager_base/local
  - vi server.conf
  - under [clustering] ensure mode is manager, replication factor is whatever is defined, and search factor is whatever is defined
  - change the pass4symkey (once enterered it will be encrypted) -> enter in keepass
  - mv the file under /opt/splunk/etc/apps
  - login into web UI
  - under clustering:manager node, ensure there are no peers configured

  - # set volumes on indexers (base configs)
-  winscp base config files: org_cluster_indexer_base, org_indexer_volume_indexes to the /tmp file on the indexer
  -  rename each to desired name ie: j_cluster_indexer_base, and j_indexer_volume_indexes : :mv org_cluster_indexer_base j_cluster_indexer_base"
  -  change ownership : chown -R splunk:splunk j_cluster_indexer_base, and j_indexer_volume_indexes
  -  in cluster_indexer_base you must change some configs in some files. Drill down into local
  -  leave app.conf,inputs.conf(for now) and indexes.conf alone
  -  in server.conf: in the [clustering] stanza, change manager uri to the ip address and port of cluster manager ie: manager_uri = https://12:213:123:12:8089, add the same pass4symmkey from cluster manager in as well. Change replication_port to ://9100. in the [license] stanza change the manager_uri to the ip address the license manager ip. ie: manager_uri = https://34:232:235:98:8089
  -  in web.conf under the [settings] stanza change startwebserver = true. For the time being. You will turn this back false later in the build.
  -  in _indexer_volume_indexes you must also change data in indexes.conf
  -  change to [volume:hot], path =/ hot #or whatever the hot directory is called change the maxVolumeDataSizeMB to the hot volume size. You must add a [volume:cold], define path to path = /cold, change the maxVolumeDataSizeMB to the cold volume size

  # Join indexer to Cluster (Peering)
  - use base config: org_cluster_indexer_base
  - go down to /opt/splunk/etc/system/local
  - edit [clustering] stanza
  - manager_uri = IP address of cluster manager:8089
  - mode=peer
  - pass4symm = the pass4sym established by the cluster manager
  # Cluster Manager Base configs
  - winscp cluster_manager_base and org_all_indexes to server
  - change ownership and rename to something appropiate
  - in org_all_indexes inside of indexes.conf change homepath to volume:hot/xindex/db # this needs to be done globally. theres a shortcut to do this in vi (enter that here) (do I need to change the coldpath to volume:cold?)
  - move org_all_indexes to manager-apps. ie: mv org_all_indexes
  - in cluster_manager_base, traverse down into server.conf. in the [clustering] stanza ensure mode = manager, replication factor =3 and search factor =2 (this is deployment dependent). define pass4symmkey here. (update keepass)

  # Change deployment server with base config
  - find base config _org_full_license_server -> push to deployment server in /opt/splunk/etc/deployment-apps (check ownership)
  - change the server.conf server to show license manager URI

  # Put org_all_deploymentclient base config on license manager, cluster manager, and search head deployer
   - move to opt/splunk/etc/apps (remember to check ownership
   - put in FQDN or IP address into targetUri (deployment server) under [target-broker:deploymentServer]

  # move org_full_license_server base config to cluster manager and deployer to /etc/apps
    - change ownership 
    edit server.conf file to add in license manager IP and port
  # move base configs related to search heads and search head clustering to indexers (forward output to indexers, and join to cluster. This is done on all search heads
    - org_all_forwarder_outputs, org_search_volume_indexes (possibly org_cluster_search_base)
    - move the above base configs to /opt/splunk/etc/apps
    - edit org_all_forwarder_outputs: add indexer ip's and replication ports(9997)
    - edit org_search_volume_indexes, vi indexes.conf, and change volumes to appropiate (ie:hot /hot, cold /cold)
  # joining search head members into the cluster
    - change to user splunk su splunk
    - move to /opt/splunk/bin
    - ./splunk init schluster-config -mgmt_uri https://ipofcurrentsearchhead -replication_port 9200 -replication_factor 2 -conf_deploy_fetch_url https://ipofDeployer:8089 -secret makeThisSameAsPass4SymmKey -schcluster_label makeThisWhateverYouWantClusterToBeNamed\
    - give admin creds
    - systemctl restart Splunkd
    - repeat on the rest of the search heads, remember to change the mgmt_uri
    - once finished, identify the captain (searchhead01)
    - change to splunk user (su splunk)
    - change to /opt/splunk/bin
    - ./splunk bootstrap schclsuter-captain -servers_list "https://ipOfEachSearchHeadInCluster:8089, https://ipofsearchHead:8089, https://ipOfSearchHead:8089"
    - enter admin creds
# add search heads to recognize the index cluster manager for access to the indexers (done on search heads)
  - go through the search heads and do the following (DO NOT RESTART BETWEEN CONFIG EDITS)
  - /opt/splunk/bin
  - ./splunk edit cluster-config -mode searchhead -master_uri https://ipOfClusterManager:8089 -secret secretidentifiedearlier
  - manuever to search head captain
  - conduct rolling restart /opt/splunk/bin/splunk rolling-restart shcluster-members
# Go to Deployer ensure searchhead cluster password in contained
  - move org_cluster_search_base to deployer through winscp (put it in /tmp then move over to /opt/splunk/etc/apps)
  - ensure that ownership is correct
  - drill down into server.conf and edit it
  - under[clustering] delete everything after clustermanager:one
  - under [clustermanager:one] change manager_uri=ipOfClusterManager:8089, change pass4symmkey to cluster managers pass4symmkey, comment outeverything in clustermanager:two
  - restart Splunkd
# Check KVstore status on search heads
  -  change user to splunk
  -  move to /opt/splunk/bin
  -  ./splunk show kvstore-status
  -  kvstore MUST have status as ready
