# GlusterFS && MinIO
A highly available and distributed storage system using GlusterFS for two-way active replication over dedicated sdb drives, integrated with MinIO Object Storage on Ubuntu. Built for seamless data synchronization, automatic self-healing, and failover resilience.

# 📌 Quick Terminology Guide
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
