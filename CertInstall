

https://www.youtube.com/watch?v=vI7466EwG7I

Basically the steps are: Create the directory /opt/splunk/etc/auth/mycerts and cd to it...
Create the private key (generate using RSA)
~/bin/splunk cmd openssl genrsa -aes256 -out CAPrivateKey.key 2048
~/bin/splunk cmd openssl req -new -key CAPrivateKey.key -out CACertificate.csr
enter all of the required information (such as the password)
Create the .pem file from the csr file
~/bin/splunk cmd openssl x509 -req -in CACertificate.csr -sha512 -signkey CAPrivateKey.key -CAcreateserial -out CaCertificate.pem -days 1095
There are now 3 files CaCertificate.csr CAPrivateKey.key and CaCertificate.pem
The CACertificate.pem is the most important file that we are working with
Generate the private certificate for the specific server
~/bin/splunk cmd openssl genrsa -aes256 -out ServerPrivKey.key 2048
Generate a csr for the specific server
~/bin/splunk cmd openssl req -new -key ServerPrivKey.key -out ServerCertificate.csr
should now have 5 files CaCertificate.csr CAPrivateKey.key and CaCertificate.pem, ServerPrivateKey.key and ServerCertificate.csr
Now, generate the ServerCertificate.pem (from ServerCertificate.csr, CaCertificate.pem, and CaPrivateKey.key)
~/bin/splunk cmd openssl x509 -req -in ServerCertificate.csr -SHA256 -CA CACertificate.pem -CAkey CAPrivateKey.key -CAcreateserial -out ServerCertificate.pem -days 1095
Bring them all together (the order is very important):
cat ServerCertificate.pem ServerPrivateKey.key CACertificate.pem > SplnkServerCertificate.pem
To see if things will work: (on a new install first do: /opt/splunk/bin/splunk enable listen 9997)
~/bin/splunk cmd openssl s_server -accept 9997 -cert SplnkServerCertificate.pem
Look for ACCEPT
Modify the inputs.conf in an APP or on etc/system/local (or create it on a new install)
[splunktcp-ssl:9997]
disabled = 0

[SSL]
serverCert = /opt/splunk/etc/auth/mycerts/SplnkServerCertificate.pem
sslPassword = <password entered for the certs>
requiredClientCert = false

In the server.conf (either in an app or /etc/system/local)
under the sslConfig stanza
sslRootCertCAPath = /opt/splunk/etc/auth/mycerts/CACertificate.pem

Copy SplnkServerCertificate.pem and CACertificate.pem to another indexer (This is a shortcut that is NOT documented) and log into the next indexer
mkdir /opt/splunk/etc/auth/mycerts (if not already created, make sure that it is owned by the splunk user)
move the uploded .pem files to the new directory
update the server.conf as in Step #9
update the inputs.conf as in Step #8
To see if things will work: (on a new install first do: /opt/splunk/bin/splunk enable listen 9997)
restart splunk
~/bin/splunk cmd openssl s_server -accept 9997 -cert SplnkServerCertificate.pem
Look for ACCEPT
FOR UNIVERSAL FORWARDERS, upload the SplnkServerCertificate.pem and CACertificate.pem to the Universal Forwarder to /opt/splunkforwarder/etc/auth/mycerts
Modify or create an outputs.conf
[tcpout]
sslPassword = <whatever password used to create the .pem files.>

under the
[tcpout:<group name.]
clientCert = /opt/splunkforwarder/etc/auth/mycerts/SplnkServerCertificate.pem
sslVerifyServerCert = true
useClientSSLCompression = true

b. Modify or create a server.conf
[sslConfig]
sslRootCAPath = /opt/splunkforwarder/etc/auth/mycerts/CaCertificate.pem

To test, you could telnet from the U/F to an indexer:
telnet <server ip> 9997
see if it connects
Restart the U/F
See if you get a connected message in Splunkd.log on the U/F
on a Searchhead
index=_internal source=*metrics*.log group=tcpin_connections
| dedup hostname |rename hostname AS Forwarder, ssl AS ssl_connection_status, fwdType AS Forwarder_Type
| table _time Forwarder Forwarder_Type version sourceIp destPort connectionType ssl_connection_status

connetionType should = cookedSSL

NOTE: To do this on a "regular" search head
In org_all_forward_outputs - there is a stanza that has a SSL entry for EACH indexer. Update accordingly in that area
