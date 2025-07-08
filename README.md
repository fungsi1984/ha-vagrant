# High-Availability with Keepalived and Vagrant

This project demonstrates how to set up a high-availability (HA) system using `keepalived` on two virtual machines managed by Vagrant. The setup consists of a master server and a backup server, with a virtual IP address that fails over from the master to the backup if the master becomes unavailable.

## Prerequisites

*   Vagrant and VirtualBox installed.
*   Two virtual machines (e.g., `server01` and `server02`) configured in the `Vagrantfile`.

## Installation

First, install `keepalived` and `libipset13` on both `server01` and `server02`:

```bash
sudo apt update
sudo apt install -y keepalived libipset13
```

## Configuration

Next, configure `keepalived` on each server by editing the `/etc/keepalived/keepalived.conf` file.

### Master Server (server01)

On `server01`, use the following configuration. This sets the server as the `MASTER` with a higher priority.

```
vrrp_instance VI_1 {
  state MASTER
  interface enp0s8 # Find your interface with `ip a`
  virtual_router_id 55
  priority 150
  advert_int 1
  unicast_src_ip 192.168.56.101 # IP of server01
  unicast_peer {
    192.168.56.102 # IP of server02
  }

  authentication {
    auth_type PASS
    auth_pass secret
  }

  virtual_ipaddress {
    192.168.56.100/24 # The virtual IP address
  }
}
```

### Backup Server (server02)

On `server02`, use the following configuration. This sets the server as the `BACKUP` with a lower priority.

```
vrrp_instance VI_1 {
  state BACKUP
  interface enp0s8 # Find your interface with `ip a`
  virtual_router_id 55
  priority 100
  advert_int 1
  unicast_src_ip 192.168.56.102 # IP of server02
  unicast_peer {
    192.168.56.101 # IP of server01
  }

  authentication {
    auth_type PASS
    auth_pass secret
  }

  virtual_ipaddress {
    192.168.56.100/24 # The virtual IP address
  }
}
```

### Configuration Notes

*   `state`: `MASTER` for the primary server, `BACKUP` for the secondary.
*   `interface`: The network interface to monitor. Use `ip a` to find the correct one.
*   `virtual_router_id`: A unique ID for the VRRP group (must be the same on both servers).
*   `priority`: The server with the higher priority becomes the master.
*   `unicast_src_ip`: The IP address of the current server.
*   `unicast_peer`: The IP address of the other server in the HA pair.
*   `virtual_ipaddress`: The shared virtual IP address.

## Verification

1.  **Enable and start the `keepalived` service on both servers:**

    ```bash
    sudo systemctl enable --now keepalived.service
    ```

2.  **Check the status of the `keepalived` service:**

    ```bash
    sudo systemctl status keepalived.service
    ```

3.  **Verify the virtual IP:**

    From either server or your host machine, ping the virtual IP address. You should get a response.

    ```bash
    ping 192.168.56.100
    ```

    You can also run `ip a` on both servers to see which one has the virtual IP address assigned to its interface.

## Failover Testing

To test the failover, stop the `keepalived` service on the `MASTER` server (`server01`):

```bash
sudo systemctl stop keepalived.service
```

Now, ping the virtual IP address again. You should still get a response, as `server02` should have taken over as the master. If you run `ip a` on `server02`, you will see the virtual IP address assigned to its interface.

To restore the original state, start the `keepalived` service on `server01` again. Since it has a higher priority, it will become the master again.
