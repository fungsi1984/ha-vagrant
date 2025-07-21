### db01
- sudo mysql
```
create database wordpress;
create user wordpress identified by "wordpress";
grant all privileges on wordpress.* to 'wordpress';
flush privileges;
exit;
```

### server01
- cd /var/tmp
- wget https://wordpress.org/latest.tar.gz
- sudo tar xf latest.tar.gz -C /var/www
- cd /var/www
- sudo chown -R www-data:www-data wordpress/
- cd wordpress/
- sudo cp -p wo^C, not sure
- ls -dl *samp*
- sudo cp -p wp-config-sample.php wp-config.php
- sudo vi wp-config.php
```
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'wordpress');
define('DB_HOST', '127.0.0.1');
```
ps -ef | grep apache
ps -ef | grep haproxy

192.168.56.2/wp-admin/install.php
fail, because there is no php-mysqli
sudo apt-get update && sudo apt-get install php-mysql
sudo nano /etc/php/<version>/apache2/php.ini 
```
extension=mysqli
extension=pdo_mysql
```


### db01
sudo mysql -p
show databases;
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
USE wordpress; 
SHOW TABLES;
SHOW TABLES FROM wordpress;
```