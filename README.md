# GlusterFS && MinIO
A highly available and distributed storage system using GlusterFS for two-way active replication over dedicated sdb drives, integrated with MinIO Object Storage on Ubuntu. Built for seamless data synchronization, automatic self-healing, and failover resilience.

📌 Quick Terminology Guide
VM1 / VM2: Virtual Machine 1 and Virtual Machine 2

- VM1_IP / VM2_IP: The actual internal IP addresses of your machines (e.g., 192.168.56.101)

- /mnt/sdb_storage/gluster_brick: The physical folder on your dedicated sdb hard drive (The Backend Brick).

- /mnt/myvol: The smart network gateway where files are read and written (The GlusterFS Mount Point).

# 🛠️ PART 1: CONFIGURING GLUSTERFS
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
5. Create and Start the Replicated Volume (Run ONLY on VM1)
```bash
# Create the volume using the brick sub-folders
sudo gluster volume create myvol replica 2 VM1_IP:/mnt/sdb_storage/gluster_brick VM2_IP:/mnt/sdb_storage/gluster_brick force

# Start the volume
sudo gluster volume start myvol

# For verification checking:
sudo gluster volume info myvol
```
6. Mount the GlusterFS Volume (Run on BOTH VMs)
This connects your local /mnt/myvol folder to the smart network layer.
- On VM1:
  ```bash
  sudo mount -t glusterfs localhost:/myvol /mnt/myvol
  ```
- On VM2:
  ```bash
  sudo mount -t glusterfs VM1_IP:/myvol /mnt/myvol
  ```

# 🧪 PART 2: THE CRITICAL CLUSTER TESTING STEPS
Before proceeding to MinIO, run these tests to verify your storage backend is completely resilient.

Test A: Normal Replication
1. On VM1, create a file:
```bash
echo "Normal sync test: Working flawlessly!" | sudo tee /mnt/myvol/normal-test.txt
```
2.) On VM2, verify the file arrived automatically:
```bash
cat /mnt/myvol/normal-test.txt
```

Test B: Failover & Self-Healing
1.) On VM2, simulate a severe crash by turning off the cluster daemon and dropping the mount:
```bash
sudo systemctl stop glusterd && sudo umount -f /mnt/myvol
```
2.) On VM1, write data while VM2 is offline (it should execute smoothly without freezing):
```bash
echo "Test" | sudo tee /mnt/myvol/fail-test.txt
```
3.) On VM2, bring the server back online and remount:
```bash
sudo systemctl start glusterd && sudo mount -t glusterfs VM1_IP:/myvol /mnt/myvol
```
4.) On VM2, wait 5–10 seconds and read the file to confirm automatic self-healing worked:
```bash
cat /mnt/myvol/fail-test.txt
```

