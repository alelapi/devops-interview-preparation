# Use Remote Filesystems and Network Block Devices

## What Are Remote Filesystems?

Remote filesystems let you access files stored on another computer over the network as if they were local. Instead of copying files back and forth, you mount the remote location and work with files directly.

Think of it like network drives in Windows, but more powerful and with different protocols for different needs.

### Why Use Remote Filesystems?

**Central Storage:**

- One server stores data, many clients access it
- Easy backups (backup one location, not many computers)
- Consistent data across multiple machines

**Resource Sharing:**

- Share files between Linux, Windows, and Mac
- Access powerful storage from lightweight clients
- Cost effective (one big storage server vs many small disks)

---

## NFS - Network File System

### What is NFS?

NFS is the native file-sharing protocol for Unix/Linux. It's fast, efficient, and designed for Linux systems to share files with each other.

**Think of it as:** Linux-to-Linux file sharing (though other OS can use it too).

**Common uses:**

- Shared home directories on office networks
- Central storage for web servers
- Shared data between multiple Linux servers
- Development teams sharing code

### NFS Versions

**NFSv3:**

- Older, widely supported
- Can use UDP or TCP
- Simpler but less features

**NFSv4:** (Recommended)

- Modern standard
- Better security (includes Kerberos support)
- Better performance
- TCP only
- Stateful (tracks connections)

---

## Setting Up NFS Server

### Installing NFS Server

```bash
# RHEL/CentOS/Rocky/Alma
dnf install nfs-utils
systemctl enable --now nfs-server

# Debian/Ubuntu
apt install nfs-kernel-server
systemctl enable --now nfs-server
```

### Configuring Exports - /etc/exports

**What it is:** This file tells NFS what directories to share and who can access them.

**Format:**

```
/path/to/share    client(options)
```

**Examples:**

```bash
# Share /data with one specific computer
/data 192.168.1.100(rw,sync,no_subtree_check)

# Share with entire network
/data 192.168.1.0/24(rw,sync,no_subtree_check)

# Share read-only with everyone
/public *(ro,sync,no_subtree_check)

# Share /home with specific computers (root on client stays root on server)
/home 192.168.1.100(rw,sync,no_subtree_check,no_root_squash)

# Multiple clients, different permissions
/data 192.168.1.100(rw) 192.168.1.0/24(ro)
```

**Common options:**

- `rw` - Read-write access
- `ro` - Read-only access
- `sync` - Sync writes immediately (safer, slower)
- `async` - Async writes (faster, less safe)
- `no_subtree_check` - Don't check subdirectories (recommended, faster)
- `no_root_squash` - Don't map root user to nobody (needed for some setups)
- `root_squash` - Map root to nobody (default, safer)

**Real-world scenario - Company file share:**

```bash
# Edit /etc/exports
vi /etc/exports

# Add share for company data
# Office computers (192.168.1.0/24) get read-write
# Remote workers (192.168.2.0/24) get read-only
/company/data 192.168.1.0/24(rw,sync,no_subtree_check) 192.168.2.0/24(ro,sync,no_subtree_check)

# Apply changes
exportfs -ra

# Verify
exportfs -v
```

### exportfs - Manage Exports

**What it does:** Applies changes to /etc/exports without restarting NFS server.

**Why use it:** Fast way to add/remove shares or change settings.

**Examples:**

```bash
# Show current exports
exportfs -v

# Reload /etc/exports (apply changes)
exportfs -ra

# Export all in /etc/exports
exportfs -a

# Unexport all
exportfs -ua

# Unexport specific directory
exportfs -u 192.168.1.100:/data

# Temporarily export (not in /etc/exports)
exportfs -o rw,sync 192.168.1.100:/tmp/test
```

**Real-world scenario:**

```bash
# Add new share to /etc/exports
echo "/backups 192.168.1.0/24(ro,sync,no_subtree_check)" >> /etc/exports

# Apply without restarting
exportfs -ra

# Verify it's exported
exportfs -v | grep backups
```

### showmount - Check NFS Exports

**What it does:** Shows what a server is exporting.

**Why use it:** Verify your exports or see what another server shares.

**Examples:**

```bash
# Show exports on local server
showmount -e localhost

# Show exports on remote server
showmount -e nfs-server.example.com

# Show all current mounts
showmount -a

# Show directories being exported
showmount -d
```

**Output example:**

```bash
showmount -e nfs-server
Export list for nfs-server:
/data      192.168.1.0/24
/home      192.168.1.100,192.168.1.101
/public    *
```

### Firewall Configuration

```bash
# RHEL/CentOS
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
```

---

## Using NFS as Client

### Installing NFS Client

```bash
# RHEL/CentOS
dnf install nfs-utils

# Debian/Ubuntu
apt install nfs-common
```

### Mounting NFS Shares

**What it does:** Connects to remote share and makes it accessible locally.

**Why use it:** Access files stored on NFS server.

**Examples:**

```bash
# Basic mount
mount nfs-server:/data /mnt/data

# Specify NFSv4
mount -t nfs4 nfs-server:/data /mnt/data

# With options
mount -t nfs -o rw,hard,intr nfs-server:/data /mnt/data

# Mount read-only
mount -t nfs -o ro nfs-server:/data /mnt/data
```

**Common mount options:**

- `hard` - Keep trying if server unavailable (recommended for important data)
- `soft` - Give up after timeout (can cause data loss)
- `intr` - Allow interruption if mount hangs
- `rsize=8192` - Read buffer size
- `wsize=8192` - Write buffer size
- `_netdev` - Wait for network before mounting (important for /etc/fstab)

**Real-world scenario - Mount company share:**

```bash
# Create mount point
mkdir -p /mnt/company-data

# Test mount first
mount -t nfs nfs-server.company.com:/data /mnt/company-data

# Check if it works
ls /mnt/company-data
df -h /mnt/company-data

# If good, make permanent in /etc/fstab
echo "nfs-server.company.com:/data /mnt/company-data nfs defaults,_netdev 0 0" >> /etc/fstab

# Test fstab
umount /mnt/company-data
mount -a
```

### Permanent NFS Mounts - /etc/fstab

**Format:**

```
server:/export  /mount/point  nfs  options  0  0
```

**Examples:**

```bash
# Basic NFS mount
nfs-server:/data /mnt/data nfs defaults,_netdev 0 0

# NFSv4 with specific options
nfs-server:/data /mnt/data nfs4 rw,hard,intr,_netdev 0 0

# With performance tuning
nfs-server:/data /mnt/data nfs rw,hard,intr,rsize=8192,wsize=8192,_netdev 0 0

# Read-only mount
nfs-server:/public /mnt/public nfs ro,_netdev 0 0
```

**Always use `_netdev`!** This tells Linux to wait for the network before mounting, preventing boot failures.

---

## CIFS/SMB - Windows File Sharing

### What is CIFS/SMB?

SMB (Server Message Block) is the file-sharing protocol used by Windows. CIFS is an older dialect of SMB. Linux can both connect to Windows shares and serve files to Windows computers.

**Think of it as:** Linux-to-Windows file sharing (and vice versa).

**Common uses:**

- Access Windows file servers from Linux
- Connect to NAS devices
- Share files between mixed Windows/Linux environments
- Access Samba shares

### Installing CIFS Client

```bash
# RHEL/CentOS
dnf install cifs-utils

# Debian/Ubuntu
apt install cifs-utils
```

### Mounting CIFS/SMB Shares

**What it does:** Connects to Windows/Samba shares.

**Why use it:** Access files on Windows servers or NAS devices.

**Examples:**

```bash
# Basic mount with username/password (not secure!)
mount -t cifs //server/share /mnt/share -o username=myuser,password=mypass

# Using credentials file (recommended)
mount -t cifs //server/share /mnt/share -o credentials=/root/.smbcreds

# With domain
mount -t cifs //server/share /mnt/share -o credentials=/root/.smbcreds,domain=COMPANY

# Specify permissions
mount -t cifs //server/share /mnt/share -o credentials=/root/.smbcreds,uid=1000,gid=1000

# Guest access (no password)
mount -t cifs //server/share /mnt/share -o guest
```

**Common options:**

- `username=user` - Username for authentication
- `password=pass` - Password (avoid, use credentials file!)
- `credentials=file` - File with username/password
- `domain=DOMAIN` - Windows domain
- `uid=1000` - Set owner of files
- `gid=1000` - Set group of files
- `file_mode=0755` - Permissions for files
- `dir_mode=0755` - Permissions for directories

### Credentials File

**Why use it:** Never put passwords directly in mount commands or /etc/fstab!

**Create credentials file:**

```bash
# Create file
vi /root/.smbcreds

# Add credentials
username=myuser
password=mypassword
domain=COMPANY

# Secure it (CRITICAL!)
chmod 600 /root/.smbcreds
chown root:root /root/.smbcreds
```

**Real-world scenario - Mount NAS share:**

```bash
# Step 1: Create credentials file
cat > /root/.nascreds << EOF
username=admin
password=SecurePassword123
EOF
chmod 600 /root/.nascreds

# Step 2: Create mount point
mkdir /mnt/nas

# Step 3: Test mount
mount -t cifs //nas.local/backup /mnt/nas -o credentials=/root/.nascreds,uid=1000,gid=1000

# Step 4: Make permanent
echo "//nas.local/backup /mnt/nas cifs credentials=/root/.nascreds,uid=1000,gid=1000,_netdev 0 0" >> /etc/fstab
```

### /etc/fstab for CIFS

```bash
# Using credentials file
//server/share /mnt/share cifs credentials=/root/.smbcreds,_netdev 0 0

# With specific permissions
//server/share /mnt/share cifs credentials=/root/.smbcreds,uid=1000,gid=1000,file_mode=0755,dir_mode=0755,_netdev 0 0

# Guest access
//server/public /mnt/public cifs guest,_netdev 0 0
```

---

## iSCSI - Network Block Devices

### What is iSCSI?

iSCSI (Internet SCSI) provides block-level access to storage over a network. Unlike NFS or SMB which share files, iSCSI makes a remote disk appear as if it's locally attached.

**Think of it as:** A hard drive that's physically in another computer but appears local.

**Key Difference:**

- **NFS/CIFS:** Share files (filesystem-level)
- **iSCSI:** Share entire disks (block-level)

**Common uses:**

- SAN (Storage Area Network) connections
- Virtual machine storage
- Database servers needing fast storage
- Clustered filesystems

**Components:**

- **Target:** The iSCSI server (provides storage)
- **Initiator:** The iSCSI client (uses storage)
- **IQN:** Unique identifier (like iqn.2024-01.com.example:server)
- **LUN:** Logical Unit Number (the actual storage unit)

---

## iSCSI Target (Server)

### Installing iSCSI Target

```bash
# RHEL/CentOS
dnf install targetcli
systemctl enable --now target

# Debian/Ubuntu
apt install targetcli-fb
systemctl enable --now target
```

### Configuring iSCSI Target with targetcli

**What it does:** Interactive tool to configure iSCSI storage.

**Why use it:** Easy way to share disks over network.

**Real-world scenario - Share a disk:**

```bash
# Enter interactive mode
targetcli

# Create backing storage (file-based, 100GB)
/backstores/fileio create disk01 /storage/disk01.img 100G

# OR use a real disk
/backstores/block create disk01 /dev/sdb

# Create iSCSI target
/iscsi create iqn.2024-01.com.example:storage01

# Create LUN (connects storage to target)
/iscsi/iqn.2024-01.com.example:storage01/tpg1/luns create /backstores/fileio/disk01

# Allow specific initiator to connect
/iscsi/iqn.2024-01.com.example:storage01/tpg1/acls create iqn.2024-01.com.example:client01

# Configure network portal
/iscsi/iqn.2024-01.com.example:storage01/tpg1/portals create 192.168.1.100

# Save and exit
saveconfig
exit
```

**View configuration:**

```bash
targetcli ls
```

### Firewall for iSCSI

```bash
firewall-cmd --permanent --add-port=3260/tcp
firewall-cmd --reload
```

---

## iSCSI Initiator (Client)

### Installing iSCSI Initiator

```bash
# RHEL/CentOS
dnf install iscsi-initiator-utils
systemctl enable --now iscsid

# Debian/Ubuntu
apt install open-iscsi
systemctl enable --now iscsid
```

### Setting Initiator Name

```bash
# Edit initiator name
vi /etc/iscsi/initiatorname.iscsi

# Set to match what target allows
InitiatorName=iqn.2024-01.com.example:client01

# Restart service
systemctl restart iscsid
```

### Connecting to iSCSI Target

**What it does:** Discovers and connects to iSCSI storage.

**Why use it:** Makes remote storage available as local disk.

**Real-world scenario:**

```bash
# Step 1: Discover available targets
iscsiadm -m discovery -t st -p 192.168.1.100

# Output shows available targets:
# 192.168.1.100:3260,1 iqn.2024-01.com.example:storage01

# Step 2: Login to target (connect)
iscsiadm -m node --login

# Step 3: Check for new disk
lsblk
# New disk appears (probably /dev/sdc)

# Step 4: Use it like any disk
# Create partition
fdisk /dev/sdc

# Create filesystem
mkfs.ext4 /dev/sdc1

# Mount
mkdir /mnt/iscsi
mount /dev/sdc1 /mnt/iscsi

# Step 5: Make permanent
echo "/dev/sdc1 /mnt/iscsi ext4 _netdev 0 0" >> /etc/fstab
```

### Managing iSCSI Sessions

```bash
# Show active sessions
iscsiadm -m session

# Show detailed session info
iscsiadm -m session -P 3

# Logout (disconnect)
iscsiadm -m node -T iqn.2024-01.com.example:storage01 -p 192.168.1.100 --logout

# Login
iscsiadm -m node -T iqn.2024-01.com.example:storage01 -p 192.168.1.100 --login

# Automatic login at boot
iscsiadm -m node -T iqn.2024-01.com.example:storage01 -p 192.168.1.100 --op update -n node.startup -v automatic

# Rescan for new LUNs
iscsiadm -m session --rescan
```

---

## Troubleshooting Remote Filesystems

### NFS Troubleshooting

**Problem: Cannot mount NFS share**

```bash
# Step 1: Check if server is exporting
showmount -e nfs-server

# Step 2: Test network connectivity
ping nfs-server
telnet nfs-server 2049

# Step 3: Check firewall
firewall-cmd --list-services

# Step 4: Try mounting with verbose
mount -v -t nfs nfs-server:/data /mnt/data

# Step 5: Check logs
journalctl -u nfs-server
dmesg | grep nfs
```

**Problem: NFS mount hangs**

```bash
# Use soft mount for testing
mount -t nfs -o soft,timeo=10 nfs-server:/data /mnt/data

# Check if server is responsive
rpcinfo -p nfs-server
```

### CIFS Troubleshooting

**Problem: Cannot mount Windows share**

```bash
# Step 1: List available shares
smbclient -L //server -U username

# Step 2: Test connection
smbclient //server/share -U username

# Step 3: Check credentials file
cat /root/.smbcreds
ls -l /root/.smbcreds  # Should be 600

# Step 4: Try with verbose
mount -v -t cifs //server/share /mnt -o credentials=/root/.smbcreds

# Step 5: Specify SMB version
mount -t cifs //server/share /mnt -o credentials=/root/.smbcreds,vers=3.0
```

**Problem: Permission denied on CIFS**

```bash
# Mount with specific UID/GID
mount -t cifs //server/share /mnt -o credentials=/root/.smbcreds,uid=1000,gid=1000,file_mode=0755,dir_mode=0755
```

### iSCSI Troubleshooting

**Problem: Cannot discover target**

```bash
# Check network
ping target-server
nc -zv target-server 3260

# Check firewall
firewall-cmd --list-ports

# Check initiator name matches
cat /etc/iscsi/initiatorname.iscsi

# Restart service
systemctl restart iscsid
```

**Problem: Lost connection**

```bash
# Check session status
iscsiadm -m session -P 3

# Restart session
iscsiadm -m node --logout
iscsiadm -m node --login

# Check logs
journalctl -u iscsid
```

---

## Performance Tips

### NFS Performance

```bash
# Increase buffer sizes
mount -o rsize=32768,wsize=32768 server:/data /mnt/data

# Use async for better performance (less safe)
mount -o async server:/data /mnt/data

# Combine options
mount -o rsize=32768,wsize=32768,async,noatime server:/data /mnt/data
```

### CIFS Performance

```bash
# Use SMB3 with multichannel
mount -t cifs //server/share /mnt -o vers=3.0,multichannel

# Increase cache
mount -t cifs //server/share /mnt -o cache=strict
```

---

## Quick Reference

### NFS

```bash
# Server
exportfs -ra                      # Reload exports
showmount -e                      # Show exports

# Client
mount nfs-server:/data /mnt       # Mount NFS share
umount /mnt                       # Unmount
```

### CIFS

```bash
# Mount
mount -t cifs //server/share /mnt -o credentials=/root/.creds

# List shares
smbclient -L //server -U user
```

### iSCSI

```bash
# Discover targets
iscsiadm -m discovery -t st -p server

# Connect
iscsiadm -m node --login

# Show sessions
iscsiadm -m session

# Disconnect
iscsiadm -m node --logout
```

### Credentials File

```bash
# Create CIFS credentials
cat > /root/.smbcreds << EOF
username=myuser
password=mypass
domain=COMPANY
EOF
chmod 600 /root/.smbcreds
```
