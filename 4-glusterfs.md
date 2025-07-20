server01
- sudo apt install xfsprogs
- sudo fdisk -l
- sudo mkfs.xfs -i size=512 /dev/sdc
- sudo mkdir -p /data/glusterfs/var-www/brick01
- sudo vi /etc/fstab
```
/dev/sdc /data/glusterfs/var-www/brick01 xfs defaults 0 0
```
- sudo mount -a
- df -h
- sudo vi /etc/hosts
```
# floating ip
192.168.56.4 vip

# private address
192.168.56.2 server01-private
192.168.56.3 server02-private
192.168.56.5 db01-private
192.168.56.6 db02-private
```
- sudo apt install glusterfs-server
- ps -ef | grep gluster
- sudo systemctl start glusterd
- sudo gluster peer probe server02-private
- sudo gluster peer status
- sudo gluster volume create var-www replica 2 transport tcp server01-private:/data/glusterfs/var-www/brick01/brick server02-private:/data/glusterfs/var-www/brick02/brick 

```
there is gonna be warning for this
vagrant@server01:~$ sudo gluster volume create var-www replica 2 transport tcp server01-private:/data/glusterfs/var-www/brick01/brick server02-private:/data/glusterfs/var-www/brick02/brick 
Replica 2 volumes are prone to split-brain. Use Arbiter or Replica 3 to avoid this. See: http://docs.gluster.org/en/latest/Administrator-Guide/Split-brain-and-ways-to-deal-with-it/.
Do you still want to continue?
 (y/n) 



```
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
- sudo mkfs.xfs -i size=512 /dev/sdc
- sudo mkdir -p /data/glusterfs/var-www/brick02
- sudo vi /etc/fstab
```
/dev/sdc /data/glusterfs/var-www/brick02 xfs defaults 0 0
```

- sudo mount -a
- df -h
- sudo vi /etc/hosts
```
# floating ip
192.168.56.4 vip

# private address
192.168.56.2 server01-private
192.168.56.3 server02-private
192.168.56.5 db01-private
192.168.56.6 db02-private


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
- likely gonna fail
- sudo systemctl start glusterd
- sudo umount -l /var/www 
- sudo umount -f /var/www 
- sudo mount -t glusterfs server01:/var-www /var/www
- sudo mkdir -p /var/www 
- sudo systemctl start apache2
- df -h
- sudo touch /var/www/from-server01
- ls -l /var/www
- sudo rm /var/www/fr*
- ls -l /var/www