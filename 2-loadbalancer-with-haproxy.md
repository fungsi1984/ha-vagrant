### notes

### server01 & server02
- sudo apt install haproxy
- sudo mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.default
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
- login with admin, pass admin
- it is gonna connected with webserver down, because we still not installing apache2 and php