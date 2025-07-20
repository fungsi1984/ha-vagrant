# **GlusterFS Deployment Guide (2025)**  
*Replicated storage for `/var/www` across two servers*

## **Prerequisites**
- 2 servers: `server01` (IP: `192.168.56.2`), `server02` (IP: `192.168.56.3`)
- Dedicated storage device (`/dev/sdb`) on each server
- Private network connectivity between nodes
- Root/sudo access

---

## **1. Storage Preparation (Both Servers)**
### **1.1 Format XFS filesystem**
```bash
sudo apt update && sudo apt install -y xfsprogs
sudo mkfs.xfs -i size=512 -n size=8192 /dev/sdb  # Optimized for Gluster
```

### **1.2 Create brick directory**  
*On server01:*
```bash
sudo mkdir -p /data/glusterfs/var-www/brick01
echo "/dev/sdb /data/glusterfs/var-www/brick01 xfs noatime,inode64 0 0" | sudo tee -a /etc/fstab
```

*On server02:*
```bash
sudo mkdir -p /data/glusterfs/var-www/brick02
echo "/dev/sdb /data/glusterfs/var-www/brick02 xfs noatime,inode64 0 0" | sudo tee -a /etc/fstab
```

### **1.3 Mount and verify**
```bash
sudo mount -a
sudo df -h /dev/sdb  # Verify mount
```

---

## **2. GlusterFS Setup**
### **2.1 Install packages (Both servers)**
```bash
sudo apt install -y glusterfs-server
sudo systemctl enable --now glusterd
```

### **2.2 Configure hosts (Both servers)**
```bash
echo -e "192.168.56.2\tserver01-private\n192.168.56.3\tserver02-private" | sudo tee -a /etc/hosts
```

### **2.3 Establish trust (From server01 only)**
```bash
sudo gluster peer probe server02-private
sudo gluster peer status  # Should show "Peer in Cluster"
```

---

## **3. Volume Creation (server01 only)**
### **3.1 Create replicated volume**
```bash
sudo gluster volume create var-www replica 2 \
  server01-private:/data/glusterfs/var-www/brick01 \
  server02-private:/data/glusterfs/var-www/brick02 \
  force
```

### **3.2 Start volume and verify**
```bash
sudo gluster volume start var-www
sudo gluster volume info  # Check "Status: Started"
```

---

## **4. Mount Configuration (Both Servers)**
### **4.1 Prepare mount point**
```bash
sudo systemctl stop apache2  # Or nginx
sudo mv /var/www{,.orig} && sudo mkdir /var/www
```

### **4.2 Add to fstab**
```bash
echo "server01-private:/var-www /var/www glusterfs defaults,_netdev,backupvolfile-server=server02-private 0 0" | sudo tee -a /etc/fstab
sudo mount -a
```

### **4.3 Restart services**
```bash
sudo systemctl start apache2
sudo systemctl restart glusterfs-server  # Ensure proper startup order
```

---

## **Verification Steps**
1. **Check volume status**  
   ```bash
   sudo gluster volume status
   ```

2. **Test failover**  
   ```bash
   # On server01:
   sudo systemctl stop glusterd
   # On server02:
   ls /var/www  # Should still be accessible
   ```

3. **Test write operations**  
   ```bash
   sudo touch /var/www/testfile_{hostname}
   ls -l /var/www  # Should appear on both servers
   ```

---

## **2025 Best Practices**
- **Security**: Consider adding TLS with `gluster volume set var-www client.ssl on`
- **Monitoring**: Install `prometheus-glusterfs-exporter` for metrics
- **Backups**: Enable snapshots with `gluster snapshot create`

---

**Notes**:  
- Replace IPs/hostnames with your actual values  
- For production: Adjust replica count based on node count  
- Test thoroughly before deploying to critical environments  

This format allows you to:  
1. Copy-paste commands directly  
2. Verify each step before proceeding  
3. Maintain clear separation between servers  

Would you like me to add any specific troubleshooting steps or post-install checks?