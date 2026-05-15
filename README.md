# GlusterFS && MinIO
A highly available and distributed storage system using GlusterFS for two-way active replication over dedicated sdb drives, integrated with MinIO Object Storage on Ubuntu. Built for seamless data synchronization, automatic self-healing, and failover resilience.

📌 Quick Terminology Guide
VM1 / VM2: Virtual Machine 1 and Virtual Machine 2

VM1_IP / VM2_IP: The actual internal IP addresses of your machines (e.g., 192.168.56.101)

/mnt/sdb_storage/gluster_brick: The physical folder on your dedicated sdb hard drive (The Backend Brick).

/mnt/myvol: The smart network gateway where files are read and written (The GlusterFS Mount Point).

🛠️ PART 1: CONFIGURING GLUSTERFS
1. Install GlusterFS Server (Run on BOTH VMs)
```bash
sudo apt update
sudo apt install -y software-properties-common
sudo apt install -y glusterfs-server
```
2. Enable and Start the Service (Run on BOTH VMs)
```bash
sudo systemctl enable --now glusterd

# For verification checking:
sudo systemctl status glusterd
```
3. Prepare the Storage Folders (Run on BOTH VMs)
Make sure your dedicated sdb drive is mounted to /mnt/sdb_storage before running these.
```bash
# Create the backend storage sub-folder (avoids mount point error)
sudo mkdir -p /mnt/sdb_storage/gluster_brick

# Create the frontend shared mount folder
sudo mkdir -p /mnt/myvol
```
4. Connect the Virtual Machines Together (Run ONLY on VM1)
Introduce the two servers to build the trusted storage pool.
```bash
sudo gluster peer probe VM2_IP

# For verification checking:
sudo gluster peer status
```
Reminder: The status MUST say Peer in Cluster (Connected). If it says disconnected, run sudo ufw disable on both VMs to stop your firewall from blocking GlusterFS ports.
