sudo apt install keepalived
sudo apt install libipset13

ip a 
	

sudo nano /etc/keepalived/keepalived.conf

### server01
vrrp_instance VI_1 {
  state MASTER
  interface enp0s8 , be aware with this, need to configure it out with ip a 
  virtual_router_id 55
  priority 150
  advert_int 1
  unicast_src_ip 192.168.56.102
  unicast_peer {
    192.168.56.102
  }

  authentication {
    auth_type PASS
    auth_pass secret
  }

  virtual_ipaddress {
    192.168.56.100/24
  }
}

### server02
vrrp_instance VI_1 {
  state BACKUP
  interface enp0s8 , be aware with this, need to configure it out with ip a 
  virtual_router_id 55
  priority 150
  advert_int 1
  unicast_src_ip 192.168.56.102
  unicast_peer {
    192.168.56.102
  }

  authentication {
    auth_type PASS
    auth_pass secret
  }

  virtual_ipaddress {
    192.168.56.100/24
  }
}
	

sudo systemctl enable --now keepalived.service
sudo systemctl status keepalived.service


ping 192.168.56.100

### server01
sudo systemctl stop keepalived.service

### server02
sudo systemctl stop keepalived.service

try ping ping 192.168.56.100, and it is not gonna works anymore