# Miscellaneous Installation Notes

- When installing Splunk using RPM:
  ```sh
  rpm -i splunk._____rpm
  ```
- If you receive an error: `useradd: cannot create directory /opt/splunk`, check if `/opt/splunk` exists.
- Switch to the Splunk user:
  ```sh
  su - splunk
  ```
- **DO NOT RUN SPLUNK AS ROOT**.
- Enable Splunk to start on reboot (must be set up as root):
  ```sh
  ./splunk enable boot-start
  ```
- Fix ownership if permissions do not align:
  ```sh
  chown -R splunk:splunk /opt/splunk
  ```
- If a PID is still running and needs to be killed:
  ```sh
  kill -9 <PID>
  ```
- Default management port for Splunk is `8089`.
- Check listening ports:
  ```sh
  ss -plnt
  ```
- The **Deployer** only pushes apps to forwarders.
- The **Cluster Manager** handles data management.
- To check if the search head cluster is up and identify the captain:
  ```sh
  ./splunk show shcluster-status
  ```
- To uninstall Splunk on RHEL:
  ```sh
  yum remove splunk
  rm -rf /opt/splunk
  ```
- The default phone-home time for `org_all_deployment_client` is 600 seconds (10 minutes) but will change after enabling universal forwarders.
- Clean the KV store:
  ```sh
  ./splunk clean kvstore --local
  ```
- Show KV store status:
  ```sh
  ./splunk show kvstore-status
  ```
- To check logs:
  ```sh
  cd /opt/splunk/var/log/splunk
  tail -f splunkd.log
  ```
- Apps like `TA-windows` will be deployed from the deployment server (depser).
- In the deployment server, under Forwarder Management, all hosts will appear under the Clients tab.
- **Never edit apps via the GUI**, always use the command line. However, you can add server classes.
- Change phone-home interval to one hour after all forwarders are deployed.
- To generate a CSR:
  ```sh
  openssl req -newkey rsa:2048 -nodes -keyout SERVER.key -out SERVER.csr -config san.cnf
  ```
- Apps that need to be deployed to search heads should be placed in `shcluster/apps` in the deployer.
- To rolling restart search head cluster members:
  ```sh
  ./splunk rolling-restart shcluster-members
  ```
- To check cluster status:
  ```sh
  ./splunk show cluster-bundle-status
  ```
- Apply cluster bundle changes:
  ```sh
  ./splunk apply cluster-bundle
  ```
- Transfer files between servers:
  ```sh
  scp -r filename splunkadmin@<IP>:<destination>
  ```
- To manually push an app change to search heads:
  ```sh
  splunk reload deploy-server
  ```
- If after deploying a forwarder you see `splunk destroying tcpoutclient during shutdown/reload` in logs, comment out `forceTimebasedautoLB` under `[tcpout:primary_indexer]` on the deployment server.
- When adding new indexes, manually define them in `org_all_indexes` on the cluster manager.
- New apps (`TAs`) should be added to the **Deployment Server**, **Deployer**, and **Cluster Manager**.
- Apply search head cluster changes from the deployer:
  ```sh
  ./splunk apply shcluster-bundle -target https://<shc-captain-IP>:8089
  ```
- Check if hot bucket replication is occurring:
  ```sh
  cd /hot
  ls -l
  ```
- Adjust storage settings once production data starts flowing:
  ```sh
  maxHOTBuckets, maxHotSpanSecs, maxWarmDBCount, coldPath.maxDataSize, frozenTimePeriodInSecs, maxTotalDataSizeMB
  ```
- Check indexer storage settings using `btool`:
  ```sh
  ~/bin/splunk btool indexes --debug list | grep frozenTimePeriodInSecs
  ```
- If seeing excessive errors in Forwarder Management, increase `phonehome` interval to 10 minutes.
- When copying `pass4symmkey` between servers, manually replace the encrypted key with its plaintext version. It will re-encrypt upon restart.

---

# Installing Splunk on a RHEL Machine

1. Turn on the machine.
2. Use **WinSCP** to move the Splunk RPM package.
3. Set file permissions:
   ```sh
   chmod 644 splunk-___.rpm
   chmod +x splunk-___.rpm
   ```
4. Install Splunk:
   ```sh
   sudo -s
   rpm -i splunk-___.rpm
   ```
5. Switch to the Splunk user and start Splunk:
   ```sh
   su - splunk
   cd /opt/splunk/bin
   ./splunk start --accept-license --answer-yes
   ```
6. Create a local admin account (**update Keepass** with credentials).
7. Stop Splunk and return to root user:
   ```sh
   ./splunk stop
   exit
   ```
8. Enable Splunk at boot:
   ```sh
   ./splunk enable boot-start -systemd-managed 1 -user splunk
   ```
9. Start Splunk:
   ```sh
   systemctl start Splunkd
   ```
10. **Enable necessary ports**:
    ```sh
    firewall-cmd --zone=public --add-port=8000/tcp --permanent
    firewall-cmd --zone=public --add-port=8089/tcp --permanent
    firewall-cmd --zone=public --add-port=9997/tcp --permanent
    firewall-cmd --zone=public --add-port=9100/tcp --permanent
    firewall-cmd --zone=public --add-port=9200/tcp --permanent
    firewall-cmd --zone=public --add-port=8191/tcp --permanent
    firewall-cmd --reload
    ```

---

# Setting Up the Cluster Manager

1. Move base config file to `/tmp`.
2. Change ownership:
   ```sh
   chown -R splunk:splunk base_config_file_manager_base
   ```
3. Move to `/opt/splunk/etc/apps`, edit `server.conf`:
   ```sh
   vi server.conf
   ```
4. Under `[clustering]`, ensure:
   - `mode = manager`
   - `replication_factor = <value>`
   - `search_factor = <value>`
   - `pass4symmkey` is set (update Keepass).

---

# Useful Splunk Queries

- Inspect database indexes:
  ```sh
  | dbinspect index=<index_name>
  ```

