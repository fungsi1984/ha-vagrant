- sudo apt install apache2 php

- when installling apache, haproxy gonna not works anymore. because port 80 get used by apache

- cd /etc/apache2/
- sudo vi ports.conf 
```
change listen from 80 into 8080
```
- sudo systemctl restart apache2
- sudo ss -tulnp | less

- check again our server01 up and server02 down
- http://192.168.56.2/haproxy/stats

- after this do same thing into server02

- check with, curl -I http://192.168.56.2
- curl -B "SRVNAME=server01" -I http://192.168.56.2

- cd /etc/apache2/sites-available/
- sudo vi wordpress.conf

```
LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" haproxy_combined

<virtualhost *:8080>
  DocumentRoot /var/www/wordpress

  <Location />
    Options -Indexes
  </Location>

  ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/wordpress_access.log haproxy_combined
</virtualhost>
```

- sudo a2ensite wordpress
- sudo a2dissite 000-default
- sudo a2enmod rewrite
- sudo systemctl reload apache2
