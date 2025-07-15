server01
- sudo apt install xfsprogs
- sudo fdisk -l
- sudo mkfs.xfs -i size=512 /dev/sdb
- sudo mkdir -p /data/glusterfs/var-www/brick01
- sudo vi /etc/fstab
```
/dev/sdb /data/glusterfs/var-www/brick01 xfs default 0 0
```
- sudo mount -a
- df -h
- sudo vi /etc/hosts
```
# floating ip
192.168.56.4

# private address
192.168.56.2
192.168.56.3
192.168.56.5
192.168.56.6
```
- sudo apt install glusterfs-server
- ps -ef | grep gluster
- sudo gluster peer probe server02-private
- sudo gluster peer status
- sudo gluster volume create var-www replica 2 transport tcp server01-private:/data/glusterfs/var-www/brick01/brick server02-private:/data/glusterfs/var-www/brick02/brick 
- sudo gluster volume start var-www
- sudo gluster volume info
- sudo vi /etc/fstab
```
server01-private:/var-www /var/www glusterfs defaults,_netdev,fetch-attempts= 5 0 0
```
- sudo systemctl stop apache2
- sudo mv /var/www{,.orig}
- ls -dl /var/ww*
- sudo mkdir /var/www
- sudo systemctl start apache2
- df -h









server02
- sudo apt install xfsprogs
- sudo fdisk -l
- sudo mkfs.xfs -i size=512 /dev/sdb
- sudo mkdir -p /data/glusterfs/var-www/brick02
- sudo vi /etc/fstab
```
/dev/sdb /data/glusterfs/var-www/brick01 xfs default 0 0
```
- sudo gluster peer status
- grep server server01 /etc/hosts
- sudo vi /etc/fstab
```
server02-private:/var-www /var/www glusterfs defaults,_netdev,fetch-attempts= 5 0 0
```
- sudo systemctl stop apache2
- sudo mv /var/www{,.orig}
- sudo mkdir /var/www
- sudo mount /var/www
- sudo systemctl start apache2
- df -h
- sudo touch /var/www/from-server01
- ls -l /var/www
- sudo rm /var/www/fr*
- ls -l /var/www