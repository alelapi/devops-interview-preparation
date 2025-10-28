# Use Remote Filesystems and Network Block Devices

## Overview
This guide covers NFS, CIFS/SMB, iSCSI, and other network storage technologies for sharing and accessing remote storage.

---

## NFS (Network File System)

### Overview
NFS is a distributed filesystem protocol that allows remote file access over a network. Commonly used in Unix/Linux environments.

### NFS Versions
- **NFSv3:** Traditional, widely compatible, UDP/TCP
- **NFSv4:** Modern, better security, stateful, TCP only, includes Kerberos support

---

## NFS Server Configuration

### Install NFS Server
```bash
# RHEL/CentOS/Rocky
dnf install nfs-utils
systemctl enable --now nfs-server

# Debian/Ubuntu
apt install nfs-kernel-server
systemctl enable --now nfs-server
```

### Configure NFS Exports

#### /etc/exports - Main Configuration File
```bash
/path/to/share client(options)
```

**Common Export Options:**
- `ro` : Read-only
- `rw` : Read-write
- `sync` : Synchronous writes (safer)
- `async` : Asynchronous writes (faster)
- `no_root_squash` : Root on client = root on server
- `root_squash` : Map root to nobody (default, safer)
- `all_squash` : Map all users to nobody
- `no_subtree_check` : Disable subtree checking (recommended)
- `anonuid=UID` : Map anonymous users to UID
- `anongid=GID` : Map anonymous users to GID

**Client Specifications:**
- `192.168.1.100` : Single IP
- `192.168.1.0/24` : Network range (CIDR)
- `*.example.com` : Wildcard hostname
- `*` : All hosts (use with caution!)

**Use Cases:**
- Share files across network
- Central storage for multiple servers
- Home directories
- Application data

**Examples:**
```bash
# Simple read-only share
/export/public 192.168.1.0/24(ro,sync,no_subtree_check)

# Read-write share for single host
/export/data 192.168.1.100(rw,sync,no_root_squash,no_subtree_check)

# Multiple clients with different permissions
/export/share 192.168.1.100(rw) 192.168.1.0/24(ro)

# Home directories
/home 192.168.1.0/24(rw,sync,no_root_squash)

# All hosts (insecure!)
/export/public *(ro,sync)
```

### exportfs - Manage NFS Exports
```bash
exportfs [options]
```
**Common Options:**
- `-a` : Export/unexport all
- `-r` : Re-export all (refresh)
- `-u` : Unexport directories
- `-v` : Verbose

**Use Cases:**
- Apply export changes without restart
- List current exports
- Temporarily unexport shares

**Examples:**
```bash
# Show current exports
exportfs -v

# Export all in /etc/exports
exportfs -a

# Re-export all (reload configuration)
exportfs -ra

# Unexport specific share
exportfs -u 192.168.1.100:/export/data

# Unexport all
exportfs -ua

# Export without /etc/exports entry
exportfs -o rw,sync 192.168.1.100:/export/test
```

### showmount - Show NFS Server Information
```bash
showmount [options] [host]
```
**Common Options:**
- `-e` : Show export list
- `-a` : Show clients and mount points
- `-d` : Show directories

**Use Cases:**
- Check server exports
- Verify client connections
- Troubleshooting

**Examples:**
```bash
# Show exports on local server
showmount -e

# Show exports on remote server
showmount -e nfs-server.example.com

# Show all current mounts
showmount -a

# Show mounted directories
showmount -d
```

### Firewall Configuration (NFSv4)
```bash
# RHEL/CentOS
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload

# Or specific ports
firewall-cmd --permanent --add-port=2049/tcp
firewall-cmd --reload
```

---

## NFS Client Configuration

### Install NFS Client
```bash
# RHEL/CentOS
dnf install nfs-utils

# Debian/Ubuntu
apt install nfs-common
```

### Mount NFS Share
```bash
mount -t nfs server:/export/path /local/mountpoint
```
**Common Mount Options:**
- `vers=4` : Use NFSv4
- `hard` : Hard mount (keep trying on failure)
- `soft` : Soft mount (timeout and error)
- `intr` : Allow interruption
- `rsize=N` : Read buffer size
- `wsize=N` : Write buffer size
- `timeo=N` : Timeout in tenths of second
- `retrans=N` : Number of retransmissions
- `nolock` : Disable file locking
- `_netdev` : Network device (wait for network)

**Use Cases:**
- Access remote files
- Mount shared storage
- Central file repository

**Examples:**
```bash
# Basic NFS mount (NFSv4)
mount -t nfs nfs-server.example.com:/export/data /mnt/nfs

# NFSv3 mount
mount -t nfs -o vers=3 server:/export /mnt/nfs

# With specific options
mount -t nfs -o rw,hard,intr,rsize=8192,wsize=8192 server:/export /mnt/nfs

# Read-only mount
mount -t nfs -o ro server:/export /mnt/nfs

# Soft mount with timeout
mount -t nfs -o soft,timeo=10,retrans=3 server:/export /mnt/nfs
```

### /etc/fstab - Persistent NFS Mounts
```bash
server:/export/path /local/mountpoint nfs options 0 0
```

**Examples:**
```bash
# NFSv4 with recommended options
nfs-server:/export/data /mnt/data nfs4 defaults,_netdev 0 0

# NFSv3 mount
nfs-server:/export/share /mnt/share nfs vers=3,hard,intr,rsize=8192,wsize=8192,_netdev 0 0

# Read-only mount
nfs-server:/export/public /mnt/public nfs ro,_netdev 0 0

# With specific user mapping
nfs-server:/export/home /home nfs rw,hard,intr,_netdev 0 0
```

### Useful NFS Client Commands

#### nfsstat - NFS Statistics
```bash
nfsstat [options]
```
**Common Options:**
- `-c` : Client statistics
- `-s` : Server statistics
- `-m` : Mounted filesystems info

**Examples:**
```bash
# Client statistics
nfsstat -c

# Show mounted NFS filesystems
nfsstat -m
```

#### Check NFS Version
```bash
# Check negotiated NFS version
nfsstat -m | grep vers

# Mount with specific version
mount -t nfs -o vers=4.2 server:/export /mnt
```

---

## CIFS/SMB (Windows File Sharing)

### Overview
CIFS (Common Internet File System) / SMB (Server Message Block) is used for Windows file sharing. Linux can both mount and serve CIFS shares.

### Install CIFS Client
```bash
# RHEL/CentOS
dnf install cifs-utils

# Debian/Ubuntu
apt install cifs-utils
```

### Mount CIFS Share
```bash
mount -t cifs //server/share /mountpoint -o options
```
**Common Options:**
- `username=user` : Username
- `password=pass` : Password (insecure!)
- `credentials=file` : Credentials file (recommended)
- `domain=DOMAIN` : Windows domain
- `uid=UID` : Local user ID
- `gid=GID` : Local group ID
- `file_mode=0755` : File permissions
- `dir_mode=0755` : Directory permissions
- `vers=3.0` : SMB protocol version
- `sec=ntlmssp` : Security mode

**Use Cases:**
- Access Windows shares
- Mount NAS devices
- Cross-platform file sharing

**Examples:**
```bash
# Basic mount with inline credentials (not recommended)
mount -t cifs //server/share /mnt/share -o username=user,password=pass

# With credentials file
mount -t cifs //server/share /mnt/share -o credentials=/root/.smbcredentials

# With domain
mount -t cifs //server/share /mnt/share -o credentials=/root/.smbcredentials,domain=DOMAIN

# With specific permissions and UID
mount -t cifs //server/share /mnt/share -o credentials=/root/.smbcredentials,uid=1000,gid=1000,file_mode=0755,dir_mode=0755

# Specify SMB version
mount -t cifs //server/share /mnt/share -o credentials=/root/.smbcredentials,vers=3.0

# Guest access (no password)
mount -t cifs //server/share /mnt/share -o guest
```

### Credentials File
```bash
# Create /root/.smbcredentials
username=myuser
password=mypassword
domain=MYDOMAIN

# Secure the file
chmod 600 /root/.smbcredentials
```

### /etc/fstab - Persistent CIFS Mounts
```bash
//server/share /mountpoint cifs credentials=/root/.smbcredentials,_netdev 0 0
```

**Examples:**
```bash
# Basic with credentials file
//fileserver/data /mnt/data cifs credentials=/root/.smbcredentials,_netdev 0 0

# With specific permissions
//fileserver/share /mnt/share cifs credentials=/root/.smbcredentials,uid=1000,gid=1000,file_mode=0755,dir_mode=0755,_netdev 0 0

# With domain
//fileserver/share /mnt/share cifs credentials=/root/.smbcredentials,domain=CORPORATE,_netdev 0 0

# SMB3
//fileserver/share /mnt/share cifs credentials=/root/.smbcredentials,vers=3.0,_netdev 0 0
```

### smbclient - Access SMB Shares
```bash
smbclient [options] //server/share
```
**Common Options:**
- `-U` : Username
- `-L` : List shares
- `-N` : No password

**Use Cases:**
- Browse Windows shares
- Test connectivity
- Transfer files

**Examples:**
```bash
# List shares on server
smbclient -L //server -U username

# Connect to share
smbclient //server/share -U username

# List shares without password (guest)
smbclient -L //server -N

# Execute command and exit
smbclient //server/share -U username -c 'ls'
```

---

## Samba Server (Serve Files via SMB/CIFS)

### Install Samba Server
```bash
# RHEL/CentOS
dnf install samba samba-client
systemctl enable --now smb nmb

# Debian/Ubuntu
apt install samba
systemctl enable --now smbd nmbd
```

### Configure Samba - /etc/samba/smb.conf
```ini
[global]
   workgroup = WORKGROUP
   server string = Samba Server
   security = user
   map to guest = Bad User

[share_name]
   path = /path/to/directory
   browseable = yes
   writable = yes
   guest ok = no
   valid users = user1, user2
   create mask = 0755
   directory mask = 0755
```

**Example Configurations:**
```ini
# Public read-only share
[public]
   path = /srv/samba/public
   browseable = yes
   writable = no
   guest ok = yes
   read only = yes

# Private read-write share
[private]
   path = /srv/samba/private
   browseable = yes
   writable = yes
   guest ok = no
   valid users = john, jane
   create mask = 0644
   directory mask = 0755

# Home directories
[homes]
   comment = Home Directories
   browseable = no
   writable = yes
   valid users = %S
```

### Samba User Management

#### smbpasswd - Manage Samba Users
```bash
smbpasswd [options] username
```
**Common Options:**
- `-a` : Add user
- `-d` : Disable user
- `-e` : Enable user
- `-x` : Delete user

**Examples:**
```bash
# Add Samba user (must be Linux user first)
useradd john
smbpasswd -a john

# Change password
smbpasswd john

# Disable user
smbpasswd -d john

# Enable user
smbpasswd -e john

# Delete Samba user
smbpasswd -x john
```

#### pdbedit - Samba User Database Tool
```bash
pdbedit [options]
```
**Common Options:**
- `-L` : List users
- `-v` : Verbose
- `-a` : Add user
- `-x` : Delete user

**Examples:**
```bash
# List all Samba users
pdbedit -L

# List with details
pdbedit -Lv

# Add user
pdbedit -a -u john
```

### Test Samba Configuration
```bash
# Test configuration file
testparm

# Verbose output
testparm -v
```

### Firewall Configuration
```bash
# RHEL/CentOS
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload

# Or specific ports
firewall-cmd --permanent --add-port=139/tcp
firewall-cmd --permanent --add-port=445/tcp
firewall-cmd --reload
```

---

## iSCSI (Internet SCSI)

### Overview
iSCSI provides block-level storage access over IP networks. Allows remote disk access as if it were local.

**Components:**
- **Target:** iSCSI server (storage provider)
- **Initiator:** iSCSI client (storage consumer)
- **IQN:** iSCSI Qualified Name (unique identifier)
- **LUN:** Logical Unit Number (storage unit)

---

## iSCSI Target (Server) Configuration

### Install iSCSI Target
```bash
# RHEL/CentOS/Rocky
dnf install targetcli
systemctl enable --now target

# Debian/Ubuntu
apt install targetcli-fb
systemctl enable --now target
```

### Configure iSCSI Target - targetcli
```bash
targetcli
```

**Use Cases:**
- Provide block storage over network
- SAN configuration
- Shared storage for clusters

**Example Configuration:**
```bash
# Enter targetcli
targetcli

# Create backing storage (file-based)
/backstores/fileio create disk1 /storage/disk1.img 10G

# Or block device
/backstores/block create disk1 /dev/sdb

# Create iSCSI target
/iscsi create iqn.2024-01.com.example:target1

# Create LUN
/iscsi/iqn.2024-01.com.example:target1/tpg1/luns create /backstores/fileio/disk1

# Set ACL (allow specific initiator)
/iscsi/iqn.2024-01.com.example:target1/tpg1/acls create iqn.2024-01.com.example:initiator1

# Configure portal (IP and port)
/iscsi/iqn.2024-01.com.example:target1/tpg1/portals create 192.168.1.100

# Save and exit
saveconfig
exit
```

**Simplified Commands:**
```bash
# All in one session
targetcli <<EOF
/backstores/fileio create disk1 /storage/disk1.img 10G
/iscsi create iqn.2024-01.com.example:target1
/iscsi/iqn.2024-01.com.example:target1/tpg1/luns create /backstores/fileio/disk1
/iscsi/iqn.2024-01.com.example:target1/tpg1/acls create iqn.2024-01.com.example:initiator1
saveconfig
exit
EOF
```

### View iSCSI Configuration
```bash
# List all configuration
targetcli ls

# Specific sections
targetcli ls /backstores
targetcli ls /iscsi
```

### Firewall Configuration
```bash
firewall-cmd --permanent --add-port=3260/tcp
firewall-cmd --reload
```

---

## iSCSI Initiator (Client) Configuration

### Install iSCSI Initiator
```bash
# RHEL/CentOS
dnf install iscsi-initiator-utils
systemctl enable --now iscsid

# Debian/Ubuntu
apt install open-iscsi
systemctl enable --now iscsid open-iscsi
```

### Configure Initiator Name
```bash
# Edit /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2024-01.com.example:initiator1
```

### iscsiadm - iSCSI Administration Tool
```bash
iscsiadm [options]
```

**Common Operations:**
- `-m discovery` : Discover targets
- `-m node` : Manage nodes
- `-m session` : Show sessions

**Use Cases:**
- Discover available targets
- Connect to iSCSI storage
- Manage iSCSI connections

**Examples:**
```bash
# Discover targets on server
iscsiadm -m discovery -t st -p 192.168.1.100

# Show discovered targets
iscsiadm -m node

# Login to target (connect)
iscsiadm -m node -T iqn.2024-01.com.example:target1 -p 192.168.1.100 --login

# Logout from target
iscsiadm -m node -T iqn.2024-01.com.example:target1 -p 192.168.1.100 --logout

# Show active sessions
iscsiadm -m session

# Show detailed session info
iscsiadm -m session -P 3

# Delete target configuration
iscsiadm -m node -T iqn.2024-01.com.example:target1 -p 192.168.1.100 -o delete

# Rescan for new LUNs
iscsiadm -m session --rescan
```

### Complete iSCSI Client Workflow
```bash
# 1. Set initiator name
echo "InitiatorName=iqn.2024-01.com.example:initiator1" > /etc/iscsi/initiatorname.iscsi

# 2. Restart iscsid
systemctl restart iscsid

# 3. Discover targets
iscsiadm -m discovery -t st -p 192.168.1.100

# 4. Login to target
iscsiadm -m node --login

# 5. Check for new disk
lsblk
fdisk -l

# 6. Use the disk (appears as /dev/sd*)
# Create partition, filesystem, mount, etc.
fdisk /dev/sdc
mkfs.ext4 /dev/sdc1
mount /dev/sdc1 /mnt/iscsi

# 7. Make persistent in /etc/fstab
echo "/dev/sdc1 /mnt/iscsi ext4 _netdev 0 0" >> /etc/fstab
```

### Automatic iSCSI Mount
```bash
# Enable automatic login
iscsiadm -m node -T iqn.2024-01.com.example:target1 -p 192.168.1.100 --op update -n node.startup -v automatic

# Disable automatic login
iscsiadm -m node -T iqn.2024-01.com.example:target1 -p 192.168.1.100 --op update -n node.startup -v manual
```

---

## Troubleshooting Remote Filesystems

### NFS Troubleshooting

#### Check NFS Service Status
```bash
systemctl status nfs-server
systemctl status rpcbind
```

#### Verify Exports
```bash
exportfs -v
showmount -e localhost
```

#### Check Firewall
```bash
firewall-cmd --list-services
firewall-cmd --list-ports
```

#### Network Connectivity
```bash
ping nfs-server
telnet nfs-server 2049
rpcinfo -p nfs-server
```

#### Mount Issues
```bash
# Check mount from client
mount | grep nfs
nfsstat -m

# Debug mount
mount -vvv -t nfs server:/export /mnt

# Check logs
journalctl -u nfs-server
dmesg | grep -i nfs
```

### CIFS/SMB Troubleshooting

#### Test Connectivity
```bash
# List shares
smbclient -L //server -U username

# Test connection
smbclient //server/share -U username
```

#### Check Credentials
```bash
# Verify credentials file
cat /root/.smbcredentials
ls -l /root/.smbcredentials  # Should be 600
```

#### Mount Debug
```bash
# Verbose mount
mount -vvv -t cifs //server/share /mnt

# Check logs
dmesg | grep -i cifs
journalctl -xe
```

#### Samba Server Issues
```bash
# Test configuration
testparm

# Check service
systemctl status smb nmb

# Check logs
tail -f /var/log/samba/log.smbd

# Verify users
pdbedit -L
```

### iSCSI Troubleshooting

#### Check Services
```bash
# Target
systemctl status target

# Initiator
systemctl status iscsid
```

#### Verify Target Configuration
```bash
targetcli ls
```

#### Check Discovery
```bash
# Discover targets
iscsiadm -m discovery -t st -p target-server

# Show nodes
iscsiadm -m node
```

#### Session Issues
```bash
# Check active sessions
iscsiadm -m session

# Detailed session info
iscsiadm -m session -P 3

# Restart session
iscsiadm -m node --logout
iscsiadm -m node --login
```

#### Network Issues
```bash
# Test connectivity
telnet target-server 3260
nc -zv target-server 3260

# Check firewall
firewall-cmd --list-all
```

#### Logs
```bash
journalctl -u iscsid
dmesg | grep -i iscsi
```

---

## Performance Considerations

### NFS Tuning
```bash
# Increase buffer sizes
mount -o rsize=32768,wsize=32768 server:/export /mnt

# Use async for better performance (less safe)
mount -o async server:/export /mnt

# In /etc/fstab
server:/export /mnt nfs rsize=32768,wsize=32768,async 0 0
```

### CIFS Tuning
```bash
# Increase cache ttl
mount -t cifs -o cache=strict //server/share /mnt

# Use multichannel (SMB3)
mount -t cifs -o vers=3.0,multichannel //server/share /mnt
```

### iSCSI Tuning
```bash
# Adjust queue depth
echo 128 > /sys/block/sdc/device/queue_depth

# Enable write cache
hdparm -W1 /dev/sdc
```

---

## Best Practices

1. **Use NFSv4** for better security and performance
2. **Always use _netdev** in fstab for network mounts
3. **Secure credentials files** with chmod 600
4. **Use credentials files** instead of inline passwords
5. **Enable firewalls** and allow only necessary ports
6. **Use hard mounts** for critical data (NFS)
7. **Test recovery procedures** for network failures
8. **Monitor network performance** and latency
9. **Use multipathing** for iSCSI redundancy
10. **Regular backups** of network-mounted data

---

## Quick Reference

### NFS
```bash
# Server
exportfs -ra                           # Reload exports
showmount -e                          # Show exports

# Client
mount -t nfs server:/export /mnt      # Mount
nfsstat -m                            # Show mounts
```

### CIFS/SMB
```bash
# Mount
mount -t cifs //server/share /mnt -o credentials=/root/.creds

# List shares
smbclient -L //server -U user
```

### iSCSI
```bash
# Discover
iscsiadm -m discovery -t st -p server

# Login
iscsiadm -m node --login

# Sessions
iscsiadm -m session
```
