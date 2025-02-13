# General guidelines for getting data into Splunk using App (Like technology add-ons)
- on Cluster manager ensure to make an index in manager-apps/org_all_indexes/indexes.conf. add to the end (look at [windows]) Just ensure that a homepath, coldpath and thawedpath are defined
- Ie: TA_windows has a default directory with NO local, and inputs.conf is not present. TA_windows is also found in Deployment Server with just the Inputs.conf found inside a local directory that was created by an administrator
- Go on the Deployment Server UI, ensure the script with the splunkforwarder install file has been pushed out
- You should be able to see the Clients that the forwarders are installed on are present under clients
- IF need be create server class that is used to group clients (ie: server classes for linux servers, and a server class for windows servers)
- On the deployment Server backend, under /opt/splunk/etc/deployment-apps | This is where you put whatever apps you want pushed out via the splunk forwarders you have installed on clients. Use winscp or whatever tools needed to get the apps there.
- Ensure permissions are correct!
- on the Deployment Server web UI, ensure after installation of the apps that enable app, and Restart Splunk are selected. (Via edit)
- **For example: with the TA_Windows app we have deployed via deployment server, we are defining the indexes in both the cluster manager in org_all_indexes at the very bottom, and on the deployment server in the TA_windows app in deployment_apps, under inputs**
