### trial and error, choose between 3
- sudo apt-get install -y mariadb-server galera
- sudo apt-get install -y mysql-server, this fail HA
- sudo apt-get install -y mysql-server mysql-shell mysql-router


### sudo apt-get install -y mysql-server, this fail HA
- cd /etc/mysql
- sudo mv my.cnf my.cnf.default
- sudo vi my.cnf
```
[mysqld]
datadir=/var/lib/mysql
user=mysql

# Path to the Galera library.
wsrep_provider=/usr/lib/libgalera_smm.so

# Cluster connection URL contains the private IPs of all nodes.
wsrep_cluster_address=gcomm://192.168.56.5,192.168.56.6

# In order for Galera to work correctly the binlog format should be ROW.
binlog_format=ROW

# MyISAM storage engine has only experimental support
default_storage_engine=InnoDB

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# SST method
wsrep_sst_method=xtrabackup

# Cluster name
wsrep_cluster_name=my_lamp_cluster

# Authentication for SST method
wsrep_sst_auth="sstuser:sstuser"

# db01 private IP address
wsrep_node_address=192.168.56.5

```

- do this with db02
```
[mysqld]
datadir=/var/lib/mysql
user=mysql

# Path to the Galera library.
wsrep_provider=/usr/lib/libgalera_smm.so

# Cluster connection URL contains the private IPs of all nodes.
wsrep_cluster_address=gcomm://192.168.56.5,192.168.56.6

# In order for Galera to work correctly the binlog format should be ROW.
binlog_format=ROW

# MyISAM storage engine has only experimental support
default_storage_engine=InnoDB

# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2

# SST method
wsrep_sst_method=xtrabackup

# Cluster name
wsrep_cluster_name=my_lamp_cluster

# Authentication for SST method
wsrep_sst_auth="sstuser:sstuser"

# db02 private IP address
wsrep_node_address=192.168.56.6
```

- sudo mysql, fail 

### sudo apt install mariadb-server galera-4

- cd /etc/mysql
- sudo mv my.cnf my.cnf.default
- sudo vi my.cnf

- we gonna rewrite from percona into mariadb galera
```
[server]
datadir=/var/lib/mysql
user=mysql

[galera]
# Path to MariaDB's Galera library (different path from Percona)
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster nodes (same as before)
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"

# Required settings for Galera
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2

# SST method (use mariabackup instead of xtrabackup)
wsrep_sst_method=mariabackup

# Cluster name (same as before)
wsrep_cluster_name="my_lamp_cluster"

# SST authentication (create this user in MariaDB)
wsrep_sst_auth="sstuser:sstuser"

# Node IP (same as before)
wsrep_node_address="192.168.56.5"

# Additional recommended settings for MariaDB
wsrep_slave_threads=4
wsrep_causal_reads=ON
wsrep_sst_receive_address="192.168.56.5:4444"
```

- sudo systemctl restart mariadb
- sudo mysql
- create user 'sstuser'@'localhost' identified by 'sstuser';
- grant reload, lock tables, replication client on *.* to 'sstuser'@'localhost';
- flush privileges;
- show status like 'wsrep_cluster%';
- exit
- sudo galera_new_cluster 



- do same thing for db02

```
[server]
datadir=/var/lib/mysql
user=mysql

[galera]
# Path to Galera library (same as node 1)
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster nodes (same list, but node order doesn't matter)
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"

# Required Galera settings (same as node 1)
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2

# SST method (same as node 1)
wsrep_sst_method=mariabackup

# Cluster name (MUST match node 1 exactly)
wsrep_cluster_name="my_lamp_cluster"

# SST authentication (same credentials as node 1)
wsrep_sst_auth="sstuser:sstuser"

# CRITICAL DIFFERENCE: This node's own IP
wsrep_node_address="192.168.56.6"

# Optional tuning (same as node 1 for consistency)
wsrep_slave_threads=4
wsrep_causal_reads=ON
wsrep_sst_receive_address="192.168.56.6:4444"  
```

- sudo systemctl restart mariadb
- ping 192.168.56.5  # From db02 to db01
- ping 192.168.56.6  # From db01 to db02
- nc -zv 192.168.56.5 4567  # Test Galera port


- not sure anymore, we gonna fresh start
```
sudo rm -rf /var/lib/mysql/*
sudo mysql_install_db --user=mysql
sudo galera_new_cluster
```
- just nuke everything 

``` 
sudo apt purge mariadb-*
sudo rm -rf /var/lib/mysql/ /etc/mysql /var/log/mysql
sudo apt autoremove
sudo apt autoclean
sudo apt --fix-broken install
sudo dpkg --configure -a
```

- again fresh start
```
sudo apt install mariadb-server galera-4
sudo nano /etc/mysql/mariadb.conf.d/60-galera.cnf
cd  /etc/mysql/mariadb.conf.d/
sudo mv 60-galera.cnf 60-galera.cnf.backup

db01 192.168.56.5
[server]
datadir=/var/lib/mysql
user=mysql

[galera]
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"
wsrep_node_address="192.168.56.5"
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
wsrep_sst_method=mariabackup
wsrep_cluster_name="my_lamp_cluster"
wsrep_sst_auth="sstuser:sstuser"
wsrep_slave_threads=4


db02 192.168.56.6
[server]
datadir=/var/lib/mysql
user=mysql

[galera]
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"
wsrep_node_address="192.168.56.6"
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
wsrep_sst_method=mariabackup
wsrep_cluster_name="my_lamp_cluster"
wsrep_sst_auth="sstuser:sstuser"
wsrep_slave_threads=4

```

- critical setup
db01
sudo systemctl stop mariadb
sudo galera_new_cluster  # Initialize new cluster

db02
sudo systemctl stop mariadb
sudo rm -rf /var/lib/mysql/*  # Clear for clean SST
sudo systemctl start mariadb

again error
Job for mariadb.service failed because the control process exited with error code.
See "systemctl status mariadb.service" and "journalctl -xeu mariadb.service" for details.

### Allow TCP ports
sudo ufw allow from 192.168.56.0/24 to any port 4567,4444,4568,3306 proto tcp

### Allow UDP port 4567 (used for Galera group communication)
sudo ufw allow from 192.168.56.0/24 to any port 4567 proto udp

- again error
sudo mysql -e "SHOW STATUS LIKE 'wsrep%';" | grep -E 'ready|connected|size'

- again just nuke everything
- start from installing
```
sudo apt-get update &&
sudo apt-get install mariadb-server galera-4

# Allow MariaDB/MySQL port
sudo ufw allow 3306/tcp

# Allow Galera Cluster communication
sudo ufw allow 4567/tcp

# Allow Incremental State Transfer (IST)
sudo ufw allow 4568/tcp

# Allow State Snapshot Transfer (SST)
sudo ufw allow 4444/tcp

sudo nano /etc/mysql/conf.d/galera.cnf

db01
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Cluster name
wsrep_cluster_name="my_galera_cluster"

# Node-specific settings
wsrep_node_address="192.168.56.5"  # Changed from "this_node_ip" to actual IP
wsrep_node_name="db01"             # Changed from generic "node1" to host-specific

# SST method (xtrabackup-v2 recommended)
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth="sst_user:sst_password"

# Tuning parameters
innodb_flush_log_at_trx_commit=0
innodb_buffer_pool_size=1G         # Adjust based on available RAM

db02
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Cluster name
wsrep_cluster_name="my_galera_cluster"

# Node-specific settings
wsrep_node_address="192.168.56.6"  # Changed from "this_node_ip" to actual IP
wsrep_node_name="db02"             # Changed from generic "node1" to host-specific

# SST method (xtrabackup-v2 recommended)
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth="sst_user:sst_password"

# Tuning parameters
innodb_flush_log_at_trx_commit=0
innodb_buffer_pool_size=1G         # Adjust based on available RAM


- morelikely fail because we have low ram
db01
[galera]
# Mandatory settings
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Cluster name
wsrep_cluster_name="my_galera_cluster"

# Node-specific settings
wsrep_node_address="192.168.56.5"  # Change per node
wsrep_node_name="db01"             # Change per node

# SST method (use mariabackup or rsync for lower memory)
wsrep_sst_method=rsync             # Changed from xtrabackup (less RAM intensive)
# wsrep_sst_auth="sst_user:sst_password" # Not needed for rsync

innodb_buffer_pool_size=128M  # Was 2GB → Too high for 1GB VM!
innodb_log_buffer_size=2M     # Reduce from default 16M
key_buffer_size=8M            # Reduce from default
query_cache_size=0            # Disable (uses too much RAM)
wsrep_slave_threads=2         # Reduce from 8 (too many for 1GB)

db01
sudo mysql
CREATE USER 'sst_user'@'localhost' IDENTIFIED BY 'sst_password';
GRANT RELOAD, PROCESS, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'sst_user'@'localhost';
FLUSH PRIVILEGES;

```

- again we nuke
sudo apt install mariadb-server
sudo nano /etc/mysql/mariadb.conf.d/60-galera.cnf


db01
[server]

[mysqld]

[galera]
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_name="my_galera_cluster"
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"
wsrep_sst_method=rsync 
wsrep_node_address="192.168.56.5"
wsrep_node_name="db01"   
binlog_format=ROW        

db02
[server]

[mysqld]

[galera]
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_name="my_galera_cluster"
wsrep_cluster_address="gcomm://192.168.56.5,192.168.56.6"
wsrep_sst_method=rsync 
wsrep_node_address="192.168.56.6"
wsrep_node_name="db02"   
binlog_format=ROW    

sudo systemctl stop mariadb
sudo galera_new_cluster
journalctl -xe
sudo mysql -u root -p -e "show status like 'wrep_cluster_size'"
sudo mysql
```
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
EXIT;
```
mysql_secure_installation
n
n
y
y
n
y

vagrant@db01:~$ sudo mysql -u root -p -e "show status like 'wsrep_cluster_size'"
Enter password: 
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+

db02
sudo systemctl start mariadb

vagrant@db01:~$ sudo mysql -u root -p -e "show status like 'wsrep_cluster_size'"
Enter password: 
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+

vagrant@db01:~$ sudo mysql -u root -p
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |


### Set ha proxy for mysql

sudo vi /etc/haproxy/haproxy.cfg

```
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

# New frontend for MySQL traffic
frontend mysql_frontend
    bind *:3306
    mode tcp
    default_backend observers

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

backend observers
    mode tcp
    balance leastconn
    option httpchk
    server db01 192.168.56.5:3306 check port 9200 inter 12000 rise 3 fall 3
    server db02 192.168.12.62:3306 check port 9200 inter 12000 rise 3 fall 3 backup
    server db03 192.168.12.63:3306 check port 9200 inter 12000 rise 3 fall 3 backup
```


### utilities 
- sudo tail -n 50 /var/log/mysql/error.log
- sudo apt purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-*
- sudo find / -name "*mysql*"
- sudo apt purge mysql-* mariadb-* percona-* galera-*
- sudo apt autoremove
- sudo apt autoclean
- sudo rm -rf /etc/mysql /var/lib/mysql /var/log/mysql
- ps aux | grep -i mysql
- sudo apt purge mariadb* galera* mysql*
- sudo rm -rf /etc/mysql /var/lib/mysql
- free -h
- sudo ss -tulnp | grep -E '4567|3306'
- check if galera works or not, sudo mysql -e "SHOW STATUS LIKE 'wsrep%';" | grep -E 'ready|cluster_size|provider'
- SHOW PROCESSLIST\G;

## more on passwords and removing users
- mysql -u root -p -e "SELECT User,Host FROM mysql.user;"
- mysql -u root -p -e "SET PASSWORD FOR 'root'@'127.0.0.1' PASSWORD('new_pwd');"
- mysql -u root -p -e "SET PASSWORD FOR 'root'@'localhost' PASSWORD('new_pwd');"
- mysql -u root -p -e "DROP USER 'root'@'%';" 
- mysql -u root -p -e "DROP USER ''@'localhost';"
- mysqladmin -u root -p flush-privileges 

## create user
- mysql -u root -p -e "GRANT USAGE ON *.* TO 'abi'@'localhost' IDENTIFIED BY 'Rover#My_1st_Dog&Not_Yours!';" 
    - 'Rover#My_1st_Dog&Not_Yours!' this gonna be our password
- mysql -u root -p -e "GRANT SELECT ON *.* TO 'abi'@'localhost';"
    - it means we could just using 'select' nothing more than that
- mysql -u root -p -e "SHOW GRANTS FOR 'abi@'localhost' \G" 
    - "SHOW" to see the privileges granted to a user
- mysql -u root -p -e "GRANT ALL ON *.* TO 'abi'@'localhost';" 
    - "ALL", gives all priviledges
- mysql -u abi -p
    - connect into server

### backup
- mysqldump --user='abi' -p \ 
wordpress > wordpress.sql 

### restore
- mysql --user='abi' -p \ 
wordpress < wordpress.sql