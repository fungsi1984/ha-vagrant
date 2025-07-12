### notes

### server01 & server02
- sudo apt install haproxy
- sudo nano /etc/haproxy/haproxy.cfg

global
    log 127.0.0.1 local0 notice
    user haproxy
    group haproxy


defaults
    log     global
    option  dontlognull
    retries 3
    option redispatch
    timeout connect  5000
    timeout client  50000
    timeout server  50000

frontend http
    bind *:80
    mode http
    option httplog
    default_backend webservers

backend webservers
    mode http
    stats enable
    stats uri /haproxy/stats
    stats auth admin:admin
    stats hide-version
    balance roundrobin
    option httpclose
    option forwardfor
    cookie SRVNAME insert
    server server01 192.168.56.2:8080 check cookie server01
    server server02 192.168.56.3:8080 check cookie server02

<!-- frontend sql
    bind *:3306
    mode tcp
    option tcplog
    default_backend dbservers -->

<!-- backend dbservers
    balance leastconn
    option httpchk
    server db01 192.168.12.61:3306 check port 9200 inter 12000 rise 3 fall 3
    server db02 192.168.12.62:3306 check port 9200 inter 12000 rise 3 fall 3 backup
    server db03 192.168.12.63:3306 check port 9200 inter 12000 rise 3 fall 3 backup -->

- sudo nano /etc/default/haproxy
```
change into ENABLED=1
```

- cd /etc/rsyslog.d/
- sudo nano udp-localhost.conf
```
$ModLoad imudp
$UDPServerRun 514
$UDPServerAddress 127.0.0.1


```

- sudo systemctl restart rsyslog
- sudo systemctl restart haproxy

- 192.168.56.3/haproxy/stats

- server01 still not connected
```
vagrant@server01:~$ ip -br -c a
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp0s3           UP             10.0.2.15/24 metric 100 fd17:625c:f037:2:17:d8ff:fe8d:76f6/64 fe80::17:d8ff:fe8d:76f6/64 
enp0s8           UP             192.168.56.2/24 192.168.56.4/24 fe80::a00:27ff:feb2:967d/64 

```