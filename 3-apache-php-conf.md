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