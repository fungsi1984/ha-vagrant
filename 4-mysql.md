trial and error, choose between 3
- sudo apt-get install -y mariadb-server galera
- sudo apt-get install -y mysql-server
- sudo apt-get install -y mysql-server mysql-shell mysql-router

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