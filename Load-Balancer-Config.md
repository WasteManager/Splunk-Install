# On a linux machine, the load balancer will user the service HAproxy for load balancing
- traverse to /etc/haproxy
- create a CAcert via this command: openssl req -new -newkey rsa:2048 -nodes -keyout mykey.key -out myrequest.csr
- send that .csr to system admin to sign with CA. you will get a .cer in return
- edit the haproxy.cfg file
  Redhat repo has this. All you need is a load balancer, install via YUM haproxy It installs in the /etc/haproxy directory
There are 2 files that are configured. The first is /etc/haproxy/haproxy.cfg, the second is /etc/haproxy/certificates/haproxy.cer

haproxy.cfg
To use, replace <IP> with actual IP address
#---------------------------------------------------------------------
# Example configuration for a possible web application. See the
# full configuration options online.
#
# https://www.haproxy.org/download/1.8/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
# to have these messages end up in /var/log/haproxy.log you will
# need to:
#
# 1) configure syslog to accept network log events. This is done
# by adding the '-r' option to the SYSLOGD_OPTIONS in
# /etc/sysconfig/syslog
#
# 2) configure local2 events to go to the /var/log/haproxy.log
# file. A line like the following can be added to
# /etc/sysconfig/syslog
#
# local2.* /var/log/haproxy.log
#
log 127.0.0.1 local2

chroot /var/lib/haproxy
pidfile /var/run/haproxy.pid
maxconn 4000
user haproxy
group haproxy
daemon

# turn on stats unix socket
stats socket /var/lib/haproxy/stats

# utilize system-wide crypto-policies
ssl-default-bind-ciphers PROFILE=SYSTEM
ssl-default-server-ciphers PROFILE=SYSTEM

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
mode http
log global
option httplog
option dontlognull
option http-server-close
option forwardfor except 127.0.0.0/8
option redispatch
retries 3
timeout http-request 10s
timeout queue 1m
timeout connect 10s
timeout client 1m
timeout server 1m
timeout http-keep-alive 10s
timeout check 10s
maxconn 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend splunk-8089
mode http
bind *:8089 ssl crt /etc/haproxy/certificates/haproxy.cer ssl-min-ver TLSv1.3
http-request add-header X-Forwarded-Proto https

#acl url_static path_beg -i /static /images /javascript /stylesheets
#acl url_static path_end -i .jpg .gif .png .css .js

#use_backend static if url_static
stats uri /haproxy?stats
default_backend splunk-8089

frontend splunk-8000
mode http
bind *:8000 ssl crt /etc/haproxy/certificates/haproxy.cer ssl-min-ver TLSv1.3
http-request add-header X-Forwarded-Proto https
#acl url_static path_beg -i /static /images /javascript /stylesheets
#acl url_static path_end -i .jpg .gif .png .css .js

#use_backend static if url_static
stats uri /haproxy?stats
default_backend splunk-8000

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
# backend static
# balance roundrobin
# server static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend splunk-8089
mode http
stick-table type ip size 2 nopurge
stick on dst
balance roundrobin
cookie SERVERID insert indirect nocache
server worker1 <IP1>:8089 #ssl verify none check cookie worker1
server worker2 <IP2>:8089 #ssl verify none check cookie worker2
server worker3 <IP3>:8089 #ssl verify none check cookie worker3

backend splunk-8000
mode http
stick-table type ip size 2 nopurge
stick on dst
balance roundrobin
cookie SERVERID insert indirect nocache
server worker1 <IP1>:8000 #ssl verify none check cookie worker1
server worker2 <IP2>:8000 #ssl verify none check cookie worker2
server worker3 <IP3>:8000 #ssl verify none check cookie worker3
