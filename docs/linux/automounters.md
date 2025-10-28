# Configure Filesystem Automounters

## Overview
Automounters (autofs) automatically mount filesystems on-demand and unmount them after a period of inactivity. This saves resources and provides dynamic mounting capabilities.

---

## Automount Concepts

### Why Use Automount?
- **On-demand mounting:** Only mount when accessed
- **Resource efficiency:** Unmount idle filesystems
- **Network resilience:** Handle network interruptions
- **Scalability:** Manage many mount points efficiently
- **User convenience:** Transparent access to resources

### Components
- **automount daemon:** Background service (autofs)
- **Master map:** Main configuration file (/etc/auto.master)
- **Map files:** Define mount points and options
- **Mount points:** Directories where filesystems are mounted

### Types of Maps
- **Direct maps:** Explicit mount points (/-)
- **Indirect maps:** Mount points under a parent directory
- **Master map:** References other maps

---

## Installing and Enabling autofs

### Install autofs
```bash
# RHEL/CentOS/Rocky
dnf install autofs

# Debian/Ubuntu
apt install autofs

# SUSE
zypper install autofs
```

### Enable and Start Service
```bash
# Enable at boot and start
systemctl enable --now autofs

# Check status
systemctl status autofs

# Restart after configuration changes
systemctl restart autofs

# Reload without disrupting existing mounts
systemctl reload autofs
```

**Use Cases:**
- Initial setup
- After configuration changes
- Troubleshooting mount issues

---

## Master Map Configuration

### /etc/auto.master - Main Configuration
```bash
mount-point map-file [options]
```

**Syntax:**
- `mount-point` : Base directory (or /- for direct maps)
- `map-file` : Path to map file or map type
- `options` : Mount options (optional)

**Map Types:**
- `file:/path/to/file` : File-based map
- `ldap:` : LDAP-based map
- `nisplus:` : NIS+ map
- `nis:` : NIS map

**Use Cases:**
- Define automount hierarchies
- Configure mount behaviors
- Set global options

**Examples:**
```bash
# Indirect map
/misc /etc/auto.misc

# Direct map
/- /etc/auto.direct

# With global options
/home /etc/auto.home --timeout=60

# Multiple maps
/misc /etc/auto.misc
/net /etc/auto.net
/- /etc/auto.direct

# With strict option (require successful mount)
/data /etc/auto.data --strict

# Browse mode (show all mount points)
/misc /etc/auto.misc --browse
```

### Master Map Options
- `--timeout=N` : Unmount after N seconds of inactivity (default 300)
- `--ghost` : Create empty directories for mount points
- `--browse` : Show all available mount points
- `--strict` : Require all mounts to succeed

---

## Indirect Maps

### Concept
Indirect maps mount filesystems under a parent directory. The parent directory acts as a base for all mounts.

### Format
```bash
key [options] location
```

**Components:**
- `key` : Subdirectory name under mount point
- `options` : Mount options (optional)
- `location` : Source (NFS, local, etc.)

### /etc/auto.misc - Example Indirect Map
```bash
# Local filesystem
cdrom   -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom

# NFS mount
data    -rw,soft,intr  nfs-server:/export/data

# Multiple NFS servers (replicated)
shared  -ro  server1:/export/shared  server2:/export/shared

# Local directory bind mount
backup  -fstype=bind  :/local/backup
```

**Use Cases:**
- NFS home directories
- Shared data directories
- Removable media
- Network shares

**Complete Example:**
```bash
# /etc/auto.master
/mnt /etc/auto.mnt

# /etc/auto.mnt
cdrom    -fstype=iso9660,ro    :/dev/cdrom
usb      -fstype=auto          :/dev/sdb1
nfs1     -rw,soft               server1:/export/data
nfs2     -rw,soft               server2:/export/share
backup   -fstype=bind          :/backup/data

# Accessing:
# ls /mnt/cdrom    → mounts /dev/cdrom
# ls /mnt/nfs1     → mounts NFS share
```

---

## Direct Maps

### Concept
Direct maps specify absolute mount points anywhere in the filesystem hierarchy.

### Master Map Entry
```bash
/- /etc/auto.direct
```

### /etc/auto.direct - Example Direct Map
```bash
# Absolute mount points
/data/projects    -rw,soft    nfs-server:/export/projects
/opt/shared       -rw,hard    nfs-server:/export/shared
/mnt/backup       -fstype=bind    :/backup/main
```

**Use Cases:**
- Mount at specific locations
- Legacy application requirements
- Fixed mount point paths

**Example:**
```bash
# /etc/auto.master
/- /etc/auto.direct

# /etc/auto.direct
/data/warehouse   -rw,soft,intr   storage:/export/warehouse
/data/analytics   -rw,soft,intr   storage:/export/analytics
/mnt/archive      -ro             archive:/export/data

# Accessing:
# ls /data/warehouse    → mounts storage:/export/warehouse
# ls /data/analytics    → mounts storage:/export/analytics
```

---

## Common Map Configurations

### NFS Automounts

#### Home Directories
```bash
# /etc/auto.master
/home /etc/auto.home

# /etc/auto.home
*    -rw,soft,intr    nfs-server:/home/&

# Explanation:
# * matches any username
# & is replaced with the matched value
# User "john" accessing /home/john → mounts nfs-server:/home/john
```

#### Data Shares
```bash
# /etc/auto.master
/data /etc/auto.data

# /etc/auto.data
projects    -rw,soft    nfs-server:/export/projects
shared      -rw,soft    nfs-server:/export/shared
archive     -ro         nfs-server:/export/archive
```

### CIFS/SMB Automounts
```bash
# /etc/auto.master
/smb /etc/auto.smb

# /etc/auto.smb
share1  -fstype=cifs,credentials=/etc/auto.creds,uid=1000  ://server/share1
share2  -fstype=cifs,credentials=/etc/auto.creds,uid=1000  ://server/share2

# /etc/auto.creds (chmod 600!)
username=myuser
password=mypass
domain=DOMAIN
```

### Local Device Automounts
```bash
# /etc/auto.master
/media /etc/auto.media

# /etc/auto.media
cdrom   -fstype=iso9660,ro,nosuid,nodev    :/dev/cdrom
usb     -fstype=auto,nodev,nosuid          :/dev/sdb1
floppy  -fstype=auto                       :/dev/fd0
```

### Bind Mounts
```bash
# /etc/auto.master
/- /etc/auto.bind

# /etc/auto.bind
/mnt/data       -fstype=bind    :/var/data
/opt/app/cache  -fstype=bind    :/cache/app
```

---

## Advanced Configurations

### Multiple Locations (Replication/Failover)
```bash
# /etc/auto.data
projects  -ro  server1:/export/projects  server2:/export/projects  server3:/export/projects

# autofs will try servers in order
# First available server is used
```

### Weighted Selections
```bash
# Prefer server1, use server2 if unavailable
projects  -ro  server1(1):/export/projects  server2(2):/export/projects

# Lower weight = higher priority
```

### Variable Substitutions
```bash
# ${key} - matched key
# ${*} - entire matched string

# /etc/auto.users
*  -rw,soft  server:/home/${key}

# /etc/auto.dept
*  -rw,soft  server:/dept/${key}/data
```

### Executable Maps
```bash
# /etc/auto.master
/dynamic  program:/etc/auto.dynamic.sh

# /etc/auto.dynamic.sh (executable script)
#!/bin/bash
# Return autofs map format
if [ "$1" = "data" ]; then
    echo "-rw,soft server:/export/data"
fi
```

### Nested Mounts
```bash
# /etc/auto.master
/mnt /etc/auto.mnt

# /etc/auto.mnt
storage  /etc/auto.storage

# /etc/auto.storage
data1   -rw,soft    server:/data1
data2   -rw,soft    server:/data2

# Results in: /mnt/storage/data1, /mnt/storage/data2
```

---

## autofs Commands and Tools

### systemctl - Service Management
```bash
systemctl start autofs          # Start service
systemctl stop autofs           # Stop service
systemctl restart autofs        # Restart (disrupts mounts)
systemctl reload autofs         # Reload config (safer)
systemctl status autofs         # Check status
systemctl enable autofs         # Enable at boot
systemctl disable autofs        # Disable at boot
```

### automount - Manual Testing
```bash
# Check configuration syntax
automount -f -v

# Run in foreground with debug
automount -f -d

# Check master map
cat /etc/auto.master
```

### Auto-generated Mount Status
```bash
# Check active mounts
mount | grep autofs
df -h | grep autofs

# Check autofs mount points
ls /proc/mounts | grep autofs
```

---

## Troubleshooting autofs

### Common Issues and Solutions

#### 1. Mounts Not Working
```bash
# Check service status
systemctl status autofs

# Verify configuration syntax
automount -f -v

# Check for typos in map files
cat /etc/auto.master
cat /etc/auto.misc

# Restart service
systemctl restart autofs

# Check logs
journalctl -u autofs -f
tail -f /var/log/messages | grep automount
```

#### 2. Permission Denied
```bash
# Check NFS export permissions on server
showmount -e nfs-server

# Verify credentials (CIFS)
cat /etc/auto.creds
ls -l /etc/auto.creds  # Should be 600

# Check mount options
# Add proper uid/gid for CIFS
-fstype=cifs,credentials=/etc/creds,uid=1000,gid=1000
```

#### 3. Mounts Not Unmounting
```bash
# Check timeout setting
grep timeout /etc/auto.master

# Force unmount
umount -l /mount/point

# Check for processes using mount
lsof /mount/point
fuser -m /mount/point

# Kill processes if necessary
fuser -k /mount/point
```

#### 4. Cannot Access Mount Point
```bash
# Verify network connectivity
ping nfs-server
showmount -e nfs-server

# Check firewall
firewall-cmd --list-all

# Test manual mount
mount -t nfs nfs-server:/export/data /mnt/test

# Check autofs logs
journalctl -u autofs --since "10 minutes ago"
```

#### 5. Configuration Not Loading
```bash
# Reload autofs
systemctl reload autofs

# Full restart (disrupts existing mounts)
systemctl restart autofs

# Check for syntax errors
automount -f -v
```

### Debug Mode
```bash
# Stop service
systemctl stop autofs

# Run in debug mode
automount -f -d -v

# In another terminal, try accessing mount
ls /mnt/data

# Observe debug output
```

### Log Analysis
```bash
# View autofs logs
journalctl -u autofs -f

# System logs
tail -f /var/log/messages | grep automount
tail -f /var/log/syslog | grep automount

# Increase logging verbosity
# Edit /etc/sysconfig/autofs (RHEL) or /etc/default/autofs (Debian)
OPTIONS="--debug"
```

---

## Performance Tuning

### Timeout Optimization
```bash
# Shorter timeout for frequently changing mounts
/misc /etc/auto.misc --timeout=60

# Longer timeout for stable mounts
/data /etc/auto.data --timeout=600

# No timeout (manual unmount only)
/critical /etc/auto.critical --timeout=0
```

### Browse Mode (Show Available Mounts)
```bash
# Enable browse mode
/misc /etc/auto.misc --browse

# Pros: Users can see available mounts
# Cons: More overhead, all maps are checked
```

### Ghost Mode (Pre-create Directories)
```bash
# Enable ghost mode
/data /etc/auto.data --ghost

# Creates empty directories for all possible mounts
# Good for applications expecting directories to exist
```

---

## Security Considerations

### Protect Credential Files
```bash
# Set restrictive permissions
chmod 600 /etc/auto.creds
chown root:root /etc/auto.creds

# Never put credentials in map files directly
# Always use credential files for CIFS/SMB
```

### Use Read-Only Where Possible
```bash
# Read-only mounts for security
archive  -ro,nosuid,nodev  server:/export/archive
```

### Disable Unnecessary Options
```bash
# Use nosuid and nodev for untrusted sources
external  -nosuid,nodev,noexec  server:/export/external
```

---

## Best Practices

1. **Use Indirect Maps:** More flexible and scalable
2. **Set Appropriate Timeouts:** Balance between resources and convenience
3. **Secure Credential Files:** Always chmod 600
4. **Use Soft Mounts:** For NFS to prevent hanging
5. **Enable Logging:** For troubleshooting
6. **Test Before Production:** Verify all mounts work
7. **Document Custom Maps:** Comment complex configurations
8. **Use Replication:** Specify multiple servers for redundancy
9. **Regular Monitoring:** Check autofs status and logs
10. **Standardize Naming:** Use consistent mount point naming

---

## Complete Working Examples

### Example 1: NFS Home Directories
```bash
# /etc/auto.master
/home /etc/auto.home --timeout=600

# /etc/auto.home
* -rw,soft,intr nfs-server.example.com:/home/&

# Usage:
# User accesses /home/john
# Automatically mounts nfs-server.example.com:/home/john
```

### Example 2: Mixed Storage Environment
```bash
# /etc/auto.master
/mnt /etc/auto.mnt --timeout=300
/- /etc/auto.direct

# /etc/auto.mnt
nfs1     -rw,soft,intr          nfs-server:/export/data1
nfs2     -rw,soft,intr          nfs-server:/export/data2
smb1     -fstype=cifs,credentials=/etc/smb.creds,uid=1000  ://fileserver/share1
backup   -fstype=bind           :/backup/main

# /etc/auto.direct
/data/warehouse  -rw,hard,intr  storage:/warehouse
/data/archive    -ro            storage:/archive

# /etc/smb.creds (chmod 600)
username=svc_mount
password=SecureP@ss123
domain=CORP
```

### Example 3: Replicated Storage
```bash
# /etc/auto.master
/data /etc/auto.data --timeout=300

# /etc/auto.data
# Try server1 first, fallback to server2, then server3
projects  -ro  server1:/projects  server2:/projects  server3:/projects
shared    -rw,soft  server1:/shared  server2:/shared
```

### Example 4: Dynamic User Mounts
```bash
# /etc/auto.master
/home /etc/auto.home
/scratch /etc/auto.scratch

# /etc/auto.home
* -rw,soft,intr,nosuid home-server:/home/&

# /etc/auto.scratch
* -rw,nosuid,nodev scratch-server:/scratch/&

# Each user gets both:
# /home/username → home-server:/home/username
# /scratch/username → scratch-server:/scratch/username
```

---

## Migration from Static Mounts

### Converting /etc/fstab to autofs

**Before (/etc/fstab):**
```bash
nfs-server:/export/data /mnt/data nfs defaults 0 0
```

**After:**
```bash
# /etc/auto.master
/mnt /etc/auto.mnt

# /etc/auto.mnt
data  -rw,soft,intr  nfs-server:/export/data

# Remove from /etc/fstab
# Restart autofs
systemctl restart autofs
```

**Benefits:**
- Mounts on-demand
- Automatic unmounting
- Better resource management

---

## Quick Reference

### Installation
```bash
# Install
dnf install autofs              # RHEL
apt install autofs              # Debian

# Enable and start
systemctl enable --now autofs
```

### Basic Configuration
```bash
# /etc/auto.master
/mount-parent /etc/auto.mapfile

# /etc/auto.mapfile
key  options  location
```

### Service Management
```bash
systemctl restart autofs        # After config changes
systemctl reload autofs         # Safer, no disruption
systemctl status autofs         # Check status
journalctl -u autofs -f         # View logs
```

### Testing
```bash
# Check syntax
automount -f -v

# Debug mode
automount -f -d

# Test access
ls /mount/point
```

### Common Map Entries
```bash
# NFS
data  -rw,soft,intr  nfs-server:/export/data

# CIFS
share  -fstype=cifs,credentials=/etc/creds  ://server/share

# Local bind
backup  -fstype=bind  :/backup/data

# Wildcard (home dirs)
*  -rw,soft  server:/home/&
```
