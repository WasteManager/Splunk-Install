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
  - to uninstall splunk on rhel: yum remove splunk (ensure that /opt/splunk is removed, if not rm -rf /splunk
  - org_all_deployment_client default phone home time is 600 seconds(will be changed after universal forwarders are enabled) may take about 10 minutes for the client to show up on the deployment server
  - ./splunk clean kvstore --local #clean kvstore
  - ./splunk show kvstore-status #show kvstore status
  - check logs: move to /opt/splunk/var/log -> tail -f /opt/splunk/var/log/splunk/splunkd.log
  - apps like TA-windows will be deployed from the deployment server(depser)
  - in deployment server, under forwarder management, all hosts will appear under the clients tab
  - never edit apps from GUI, always edit from command line. But you can add server_classes
  - change phone home to something like an hour after all your forwarders are pushed out (in the deployment server[depser]) go to deployment_apps under deploymentclient.conf
  - TO generate csr do the following command: openssl req SERVER.csr -newkey rsa:2048 -nodes -keyout SERVER.key san.cnf
  - shcluster/apps in deploymon(deployer) is the location you want to put apps you want to push out to search head cluster
     members
  - To rolling restart from search head cluster captain: as splunk in the /bin dir: ./splunk rolling-restart shcluster-members
  - to show shcluster status from search head captain: ./splunk show shcluster-status
  - to show cluster status: ./splunk show cluster-bundle-status
  - after making changes in cluster manager, apply cluster bundle via cmd line. ./splunk apply cluster-bundle
  -  to move files from one server to another from cmd line: scp -r filname splunkadmin(the servers admin account)@ipaddress:/(location where you want it sent). ie: scp -r frogfile.config splunkadmin@22.9.123.43:/tmp
  -  creating a new serverclass will push app changes out to search heads, if not you can manually push it using this cmd: splunk reload deploy-server
  -  on deployment
  -  to remove an app from an deployment use the gui in deployment server. unselect, save.
  -  if after pushing out forwarder, and you tail the log and you get an error that says something along the lines of "splunk destroying tcpoutclient during shutdown/reload"(this is found in info NOT in error), go to your deployment server. Look for a [tcpout:primary_indexer] and find forceTimebasedautoLB. Comment this out.
  -  When adding apps that need new indexes, go to cluster manager(backend), into manager apps, manually add index definition to org_all_indexes(this is a base config). You can copy and past from an older index like [os] or [windows] just replace the name with whatever you want the app to send its data too
  -  Add new apps(TAs) to deployment server(only need inputs.conf THATS IT), deployer, Cluster manager
  -  apply shcluster changes to search heads from deploy: ./splunk apply shcluster-bundle -target https://ipofshcpatain:8089
  -  To check whether or not hot bucket replication is occuring on indexes: Go onto the backend of INDEXER, cd /hot, contained within are the indexes, traverse into the /db/ then ls -l, this will show up the dates that a hot bucket is being created and rolled over
  -  ONCE data starts coming in(production level volume): adjust the follows config lines found in org_all_indexes in a each individual index: maxHOTBuckets(this is hot to warm), maxHotSpanSecs(probably okay), maxWarmDBCount(warm to cold), coldPath.maxDataSize(cold to frozen), frozenTimePeriodInSecs(), maxTotalDataSizeMB(overrides everything, if data exceeds this size, it will start sending data to frozen)
  -  Configure frozen path: On the cluster manager, org_all_indexes:define the frozenTimePeriodInSecs on all indexes
  -  check maxWarmDBCOUNT, maxHotBuckets, maxHotSpanSecs on indexer using btool
  -  btool search example: (as user: splunk) ~/bin/splunk btool indexes --debug list | grep frozenTimePeriodInSecs
  -  if on the deployment server in forwarder management you are seeing alot of error and problems try moving phonehome up to 10 minutes (currently at 5 minutes)
  -  
  -  
# Installing Splunk on built out RHEL Machine
  - turn on machine
  - Use winscp to move splunk rpm to splunk machine
  - chmod 644 splunk-___.rpm
  - chmod +x splunk-___.rpm
  - sudo -s
  - rpm -i splunk-___.rpm
  - su - splunk 
  - cd /opt/splunk/bin
  - ./splunk start --accept-license --answer-yes
      -   - Will be prompted to create a local admin account: use admin and easy to remember password(update keepass!!)
          - GO UPDATE THE KEEPASS
  - ./splunk stop
  - exit (go back to root user)
  - cd /opt/splunk/bin
  - ./splunk enable boot-start -systemd-managed 1 -user splunk
  - systemctl start Splunkd
  - when specifying installs for, /cold partiton is only needed for indexers
  - # Enable correct ports
  - ensure ports are open (in powershell tnc -p 8000 #target ip address)
  - sudo -s
  - firewall-cmd --zone=public --add-port=8000/tcp --permanent
  - firewall-cmd --zone=public --add-port=8089/tcp --permanent (or use ^8000^8089)
  - firewall-cmd --zone=public --add-port=9997/tcp --permanent (or use ^8089^9997)
  - firewall-cmd --zone=public --add-port=9100/tcp --permanent #9100 being used for replication port for indexers
  - firewall-cmd --zone=public --add-port=9200/tcp --permanent #(Replication across search heads only)
  - firewall-cmd --zone=public --add-port=8191/tcp --permanent #(critical for kvstore[where clusterd knowledged objects to replicated too])
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
  - winscp cluster_manager_base and org_all_indexes to cluster manager server in manager-apps
  - change ownership and rename to something appropiate
  - in org_all_indexes inside of indexes.conf change homepath to volume:hot/xindex/db # this needs to be done globally. theres a shortcut to do this in vi (colon %s/string you want changed/thing you want string changed to/g # :%s/frogsarelame/frogsarecool/g) (do I need to change the coldpath to volume:cold?)
  - move org_all_indexes to manager-apps. ie: mv org_all_indexes
  - in cluster_manager_base, traverse down into server.conf. in the [clustering] stanza ensure mode = manager, replication factor =3 and search factor =2 (this is deployment dependent). define pass4symmkey here. (update keepass)

  # Change deployment server with base config
  - find base config _org_full_license_server -> push to deployment server in /opt/splunk/etc/deployment-apps (check ownership)
  - change the server.conf server to show license manager URI

  # Put org_all_deploymentclient base config on license manager, cluster manager, and search head deployer
   - move to opt/splunk/etc/apps (remember to check ownership deploymentclient.conf
   - put in FQDN or IP address into targetUri (deployment server) under [target-broker:deploymentServer]
   - uncomment out phone home line

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
    - ./splunk init shcluster-config -mgmt_uri https://ipofcurrentsearchhead -replication_port 9200 -replication_factor 2 -conf_deploy_fetch_url https://ipofDeployer:8089 -secret makeThisSameAsPass4SymmKey -schcluster_label makeThisWhateverYouWantClusterToBeNamed\
    - give admin creds
    - systemctl restart Splunkd
    - repeat on the rest of the search heads, remember to change the mgmt_uri
    - once finished, identify the captain (searchhead01)
    - change to splunk user (su splunk)
    - change to /opt/splunk/bin
    - ./splunk bootstrap shcluster-captain -servers_list "https://ipOfEachSearchHeadInCluster:8089, https://ipofsearchHead:8089, https://ipOfSearchHead:8089"
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

# Base Configs needed per Server
- Deployment Server: org_all_forwarder_outputs, org_cluster_search_base, org_full_license_server -> into /opt/splunk/etc/apps
- License Manager: org_all_deployment_client, org_all_forwarder_outputs -> opt/splunk/etc/apps
- Deployer: org_all_forwarder_outputs -> org_all_deployment_client -> /opt/splunk/etc/apps, org_full_license_server -> /opt/splunk/etc/deployment-apps
- cluster manager: org_all_deployment_client, org_all_forwarder_outputs, org_full_license_server, j_cluster_manager_base -> /opt/splunk/etc/apps
- indexers: j_cluster_indexer_base, j_indexer_volume_indexes -> /opt/splunk/etc/apps
- search heads: org_all_forwarder_outputs, org_search_volume_indexes -> /opt/splunk/etc/apps

# Base config tidbits
- org_indexer_volume_indexes :defines hot and cold paths. use ls -l /hot to ensure buckets are filling and are replicating
- org_cluster_indexer_base: defines pass4symm key to talk to cluster manager, and defines where the license manager lives
- org_all_deployment_client: ensures whatever machine this is put on connects to deployment server
- 
# On Search Head Cluster Deployer
- under /opt/splunk/etc/system/local
- vi server.conf
- ensure [shclustering] stanza exists, if not create it
- under the stanza add pass4SymmKey = whateverthesearchheadclusterpasswordwas
- :wq
- restart
- There is a base config with that stanza, org_cluster_search_base (alternativey you can move this into apps, and edit like above)
# add things on deployment server that need to be pushed out to non clustered splunk servers (cluster manager, license manager, deployer)
# encrypt all servers 
  - (looking for 2 certs in /opt/etc/auth/mycert[this must be created manually] CACertificate.pem and SplnkServerCertificate.pem. These will be generated using the Certinstall .md. It does not matter where you generate .pem(in regards to which server), then scp the generated .pem files to all servers.
# set up license manager as search peer (for license admin purposes)
# check again that transparent huge pages were disabled on all spl servers
# Deployment server 
- in opt/splunk/apps/splunkdeploymentserverconfig, create local dir, cp outputs.conf into new local dir, edit it to indexandforward
- # need org_all_search_volumes on each search head whether its a clustered member or non-clustered
# set up monitoring console
  - login UI
  - go to monitoring console
  - know index cluster names and search head cluster name (jcluster shcluster01 )
  - add all instances of search peers: (list of all peers) to distributed search (add search peers) in the monitoring console server (NOT clustered INDEXES) search heads, deployer, deployment server, license manager, clsuter manager
  - mon console -> settings -> general setup -> mode : distributed, save
  - confirm following: ensure unqiue values are show in each column (ie: server roles, edit these to match their intended roles)
  - GO to cluster manager and add deployment monitor to search peer (in distributed search from the UI)
  - *figure out how to maintain persistence in server settings in monitoring console after restart*

# setting up webUI certs on search heads (where you want users to access via UI)
- move base config org_all_search_base to deployer inside of /opt/splunk/etc/shcluster/apps
- delete all other configs aside from web.conf
- edit web.conf change privkeypath to privkeyPath = /opt/splunk/etc/auth/mycerts/splunkPrivKeyWebUI.key, serverCert = /opt/splunk/etc/auth/mycerts/splunkwebUI.pem
- uncomment out useSSL
- deploy out to search heads (wait)
- You may need to generate some certs and have them signed by CA (give sysadmins )
- openssl genersa -aes256-out splunkPrivateKey.key 2048
- openssl -new -key splunkPrivateKey.key -out splunk.csr (give this one to sysadmins to sign)
- decrypt the splunkPrivateKey.key -> openssl rsa -in splunkPrivateKey.key -out splunkPrivKeywebUI.key
- use the san.cnf file to generate csr: openssl req -newkey rsa:2048 -keyout splunkwebUI.key -out splunkwebUI.csr -config san.cnf (give THIS to sysadmin, ensure you edit the san.cnf(the fqdn) file before you generate the csr)

# Useful spl queries
| dbinspect index= whateverindexyouarecuriousabout

