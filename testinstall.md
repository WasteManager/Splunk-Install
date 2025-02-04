# Miscellaneous Installation Notes

- When installing Splunk using RPM:
  ```bash
  rpm -i splunk.<version>.rpm
  ```
  - If you get the error `useradd: cannot create directory /opt/splunk`, check if `/opt/splunk` exists.
- Switch to the Splunk user:
  ```bash
  su - splunk
  ```
- **DO NOT RUN SPLUNK AS ROOT.**
- Enable Splunk to start at reboot (must be set up as root):
  ```bash
  ./splunk enable boot-start -systemd-managed 1 -user splunk
  ```
- Fix permissions if needed:
  ```bash
  chown -R splunk:splunk /opt/splunk
  ```
- Kill Splunk process if it is stuck:
  ```bash
  kill -9 <PID>
  ```
- **Important Ports:**
  - 8089: Default management port for Splunk.
  - Use `ss -plnt` to check listening ports.
- **Roles in Splunk Deployment:**
  - Deployer only pushes apps to forwarders.
  - Cluster Manager handles data management.
- Check search head cluster status:
  ```bash
  ./splunk show shcluster-status
  ```
  - Displays if the search head cluster is up and who the captain is.
- **Uninstall Splunk on RHEL:**
  ```bash
  yum remove splunk
  ```
  - Ensure `/opt/splunk` is removed:
  ```bash
  rm -rf /opt/splunk
  ```
- Default **phone home** time for `org_all_deployment_client` is **600 seconds** (10 minutes).

---

## Best Case Installation Steps

1. Turn on the machine.
2. Use **WinSCP** to move the Splunk RPM to the machine.
3. Set correct permissions:
   ```bash
   chmod 644 splunk-<version>.rpm
   chmod +x splunk-<version>.rpm
   ```
4. Switch to root:
   ```bash
   sudo -s
   ```
5. Install Splunk:
   ```bash
   rpm -i splunk-<version>.rpm
   ```
6. Switch to the Splunk user:
   ```bash
   su - splunk
   ```
7. Navigate to Splunk bin directory:
   ```bash
   cd /opt/splunk/bin
   ```
8. Start Splunk with license acceptance:
   ```bash
   ./splunk start --accept-license --answer-yes
   ```
   - Create a local admin account (update **Keepass** with credentials).
9. Stop Splunk:
   ```bash
   ./splunk stop
   ```
10. Exit back to root user:
    ```bash
    exit
    ```
11. Enable Splunk to start on boot:
    ```bash
    ./splunk enable boot-start -systemd-managed 1 -user splunk
    ```
12. Start Splunk service:
    ```bash
    systemctl start Splunkd
    ```

---

## Firewall Configuration (Ensure Ports are Open)

```bash
sudo firewall-cmd --zone=public --add-port=8000/tcp --permanent
sudo firewall-cmd --zone=public --add-port=8089/tcp --permanent
sudo firewall-cmd --zone=public --add-port=9997/tcp --permanent
sudo firewall-cmd --zone=public --add-port=9100/tcp --permanent  # Replication port for indexers
sudo firewall-cmd --zone=public --add-port=9200/tcp --permanent  # Replication across search heads
sudo firewall-cmd --zone=public --add-port=8191/tcp --permanent  # KV Store communication
sudo firewall-cmd --reload  # Apply changes
```

---

## Cluster Manager Setup

1. Log into the Cluster Manager backend.
2. Move base config file to `/tmp` using **WinSCP**.
3. Change ownership:
   ```bash
   chown -R splunk:splunk base_config_file_manager_base
   ```
4. Rename config file (optional):
   ```bash
   mv base_config_file_manager_base J_base_config_file_manager_base
   ```
5. Modify `server.conf`:
   ```bash
   vi /opt/splunk/etc/apps/j_base_config_file_manager_base/local/server.conf
   ```
   - Ensure `[clustering]` settings:
     ```ini
     mode = manager
     replication_factor = <value>
     search_factor = <value>
     pass4symkey = <your_secret>
     ```
   - Save and encrypt **pass4symkey**.
6. Move the config file under `/opt/splunk/etc/apps`.
7. Log in to the **Splunk Web UI** and verify no peers are configured under **Clustering > Manager Node**.

---

## Configure Indexers

1. Transfer base config files to **/tmp**:
   ```bash
   scp org_cluster_indexer_base org_indexer_volume_indexes <indexer_ip>:/tmp
   ```
2. Rename and change ownership:
   ```bash
   mv org_cluster_indexer_base j_cluster_indexer_base
   mv org_indexer_volume_indexes j_indexer_volume_indexes
   chown -R splunk:splunk j_cluster_indexer_base j_indexer_volume_indexes
   ```
3. Modify `server.conf` on indexers:
   ```ini
   [clustering]
   manager_uri = https://<Cluster_Manager_IP>:8089
   pass4symkey = <your_secret>
   replication_port = 9100
   ```
4. Modify `indexes.conf` in `j_indexer_volume_indexes`:
   ```ini
   [volume:hot]
   path = /hot
   maxVolumeDataSizeMB = <hot_volume_size>
   
   [volume:cold]
   path = /cold
   maxVolumeDataSizeMB = <cold_volume_size>
   ```

---

## Joining Indexers to the Cluster (Peering)

1. Navigate to system local config:
   ```bash
   cd /opt/splunk/etc/system/local
   ```
2. Edit `[clustering]` settings:
   ```ini
   manager_uri = https://<Cluster_Manager_IP>:8089
   mode = peer
   pass4symmkey = <your_secret>
   ```

---

## Search Head Cluster Setup

1. Initialize cluster configuration:
   ```bash
   ./splunk init shcluster-config -mgmt_uri https://<SearchHead_IP>:8089 -replication_port 9200 -replication_factor 2 -conf_deploy_fetch_url https://<Deployer_IP>:8089 -secret <your_secret> -shcluster_label <Cluster_Name>
   ```
2. Repeat for all search heads, changing `mgmt_uri` accordingly.
3. Identify the captain (`searchhead01`).
4. Bootstrap the captain:
   ```bash
   ./splunk bootstrap shcluster-captain -servers_list "https://<SH1>:8089, https://<SH2>:8089, https://<SH3>:8089"
   ```
5. Perform a **rolling restart** of search head members:
   ```bash
   ./splunk rolling-restart shcluster-members
   ```

---

This document serves as a clean and structured guide for Splunk installation and configuration. Let me know if you need further refinements! 🚀

