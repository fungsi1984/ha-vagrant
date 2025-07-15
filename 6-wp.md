- sudo mysql
```
create database wordpress;
create user wordpress identified by "wordpress";
grant all privileges on wordpress.* to 'wordpress';
flush privileges;
exit;
```
- cd /var/tmp
- wget https://wordpress.org/latest.tar.gz
- sudo tar xf latest.tar.gz -C /var/www
- cd /var/www
- sudo chown -R www-data:www-data wordpress/
- cd wordpress/
- sudo cp -p wo^C
- ls -dl *samp*
- sudo cp -p wp-config-sample.php wp-config.php
- sudo vi wp-config.php
```
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'wordpress');
define('DB_HOST', '127.0.0.1');
```