# Configure Filesystem Automounters

## What are Automounters?

Automounters automatically mount filesystems when you access them, and automatically unmount them when they're not being used. Think of them as smart mounting - the system handles the mounting/unmounting for you.

### Why Use Automounters?

**Without automount:**

- Mount 20 network shares at boot
- Uses resources even if not needed
- Boot takes longer
- More points of failure

**With automount:**

- Mount on-demand (only when accessed)
- Unmount when idle (free resources)
- Faster boot
- More resilient to network issues

**Common uses:**

- User home directories on networks
- Shared company data
- USB/CD-ROM auto-mounting
- NFS shares that aren't always needed
- Mounting many filesystems efficiently

---

## How Automount Works

**Traditional mounting:**

```bash
# You manually mount
mount server:/home/john /home/john

# Must manually unmount
umount /home/john
```

**Automounting:**

```bash
# You just access the directory
cd /home/john

# System automatically:
# 1. Detects access
# 2. Mounts server:/home/john
# 3. Gives you access

# After 5 minutes of inactivity:
# - System automatically unmounts
```

### Components

**Master Map** (`/etc/auto.master`):

- Main configuration
- Points to other maps
- Sets global options

**Map Files**:

- Define what to mount and where
- Can be files, LDAP, NIS, etc.

**autofs Service**:

- The daemon that does the work
- Monitors access to mount points
- Handles mounting/unmounting

---

## Installing autofs

```bash
# RHEL/CentOS/Rocky
dnf install autofs
systemctl enable --now autofs

# Debian/Ubuntu
apt install autofs
systemctl enable --now autofs

# Check status
systemctl status autofs
```

---

## Master Map - /etc/auto.master

**What it is:** The main configuration file that tells autofs where to look for mount definitions.

**Format:**

```
mount-point  map-file  [options]
```

**Examples:**

```bash
# Simple indirect map
/misc /etc/auto.misc

# Direct map (special mount point /-)
/- /etc/auto.direct

# With timeout (unmount after 60 seconds)
/home /etc/auto.home --timeout=60

# Multiple maps
/misc /etc/auto.misc
/net /etc/auto.net
/- /etc/auto.direct
```

**Common options:**

- `--timeout=N` - Unmount after N seconds idle (default 300)
- `--ghost` - Create empty directories for mount points
- `--browse` - Show all available mounts (uses more resources)

---

## Indirect Maps - Most Common Type

### What are Indirect Maps?

With indirect maps, all mounts appear under a parent directory. The parent is defined in the master map, and the map file defines subdirectories.

**Example:**

- Master map says: `/data` uses `/etc/auto.data`
- Map file says: `projects` mounts from `server:/projects`
- Result: Accessing `/data/projects` automatically mounts `server:/projects`

### Creating Indirect Maps

**Real-world scenario - Company file shares:**

```bash
# Step 1: Edit master map
vi /etc/auto.master

# Add line:
/company /etc/auto.company --timeout=300

# Step 2: Create map file
vi /etc/auto.company

# Add shares:
projects    -rw,soft,intr    nfs-server:/export/projects
docs        -ro,soft         nfs-server:/export/documentation  
shared      -rw,soft,intr    nfs-server:/export/shared

# Step 3: Reload autofs
systemctl reload autofs

# Step 4: Test
ls /company/projects
# Automatically mounts nfs-server:/export/projects!

# Step 5: Check mount
df -h | grep projects
mount | grep projects

# After 5 minutes of inactivity, it auto-unmounts
```

### Map File Format

```
key  [options]  location
```

**Components:**

- **key:** Subdirectory name (becomes /mount-point/key)
- **options:** Mount options (optional)
- **location:** What to mount

**Examples:**

```bash
# NFS mount
data    -rw,soft,intr    server:/export/data

# Multiple NFS servers (failover)
backup  -ro    server1:/backup  server2:/backup  server3:/backup

# Local device
cdrom   -fstype=iso9660,ro    :/dev/cdrom

# Bind mount (mount local directory)
archive -fstype=bind    :/backup/archive
```

---

## Direct Maps - Specific Paths

### What are Direct Maps?

Direct maps let you mount to any specific path, not just under a parent directory.

**Example:**

- Want to mount at `/data/warehouse` (not `/something/warehouse`)
- Use direct map with `/data/warehouse` as the path

### Creating Direct Maps

**Real-world scenario - Specific locations:**

```bash
# Step 1: Edit master map
vi /etc/auto.master

# Add direct map line:
/- /etc/auto.direct

# Step 2: Create direct map
vi /etc/auto.direct

# Add mounts with full paths:
/data/warehouse    -rw,soft,intr    storage:/export/warehouse
/data/analytics    -rw,soft,intr    storage:/export/analytics
/opt/shared        -ro              fileserver:/export/tools
/mnt/archive       -ro              backup:/export/archive

# Step 3: Reload
systemctl reload autofs

# Step 4: Test
ls /data/warehouse
# Mounts automatically!
```

---

## Common Configurations

### User Home Directories

**Scenario:** Network home directories. When john logs in, their home mounts automatically.

```bash
# Master map
/home /etc/auto.home

# Map file (/etc/auto.home)
*    -rw,soft,intr    nfs-server:/home/&

# How it works:
# User "john" accesses /home/john
# * matches "john"
# & is replaced with "john"
# Result: mounts nfs-server:/home/john to /home/john
```

**Full example:**

```bash
# Step 1: Master map
echo "/home /etc/auto.home --timeout=600" >> /etc/auto.master

# Step 2: Map file
cat > /etc/auto.home << 'EOF'
*    -rw,soft,intr,rsize=8192,wsize=8192    homeserver:/home/&
EOF

# Step 3: Reload
systemctl reload autofs

# Now when any user logs in, their home auto-mounts!
```

### Multiple Data Shares

**Scenario:** Company has several shared drives.

```bash
# Master map
/shares /etc/auto.shares

# Map file
vi /etc/auto.shares

projects    -rw,soft    proj-server:/projects
finance     -rw,soft    fin-server:/finance
hr          -ro,soft    hr-server:/human-resources
marketing   -rw,soft    mkt-server:/marketing

# Result:
# /shares/projects → proj-server:/projects
# /shares/finance  → fin-server:/finance
# /shares/hr       → hr-server:/human-resources
# /shares/marketing → mkt-server:/marketing
```

### Windows (CIFS) Shares

**Scenario:** Mount Windows file servers.

```bash
# Master map
/windows /etc/auto.windows

# Map file
vi /etc/auto.windows

share1  -fstype=cifs,credentials=/root/.smbcreds,uid=1000,gid=1000  ://fileserver/share1
share2  -fstype=cifs,credentials=/root/.smbcreds,uid=1000,gid=1000  ://fileserver/share2

# Credentials file
cat > /root/.smbcreds << EOF
username=domain\\user
password=SecurePass123
domain=COMPANY
EOF
chmod 600 /root/.smbcreds

# Reload
systemctl reload autofs
```

### Removable Media

**Scenario:** Auto-mount USB drives and CDs.

```bash
# Master map
/media /etc/auto.media

# Map file
vi /etc/auto.media

cdrom   -fstype=iso9660,ro,nosuid,nodev    :/dev/cdrom
usb     -fstype=auto,nodev,nosuid          :/dev/sdb1
```

---

## Advanced Features

### Wildcards and Variables

**Using wildcards:**

```bash
# In auto.home
*    -rw    server:/home/&

# Matches any username
# & is replaced with matched value
```

**Variables available:**

- `${key}` - The matched key
- `${*}` - Everything matched
- `$USER` - Current user (in some contexts)

**Example with subdirectories:**

```bash
# Map file
*/*  -rw,soft  server:/dept/${0}

# Accessing /data/sales/reports
# Mounts: server:/dept/sales/reports
```

### Multiple Server Failover

**Scenario:** Use backup servers if primary fails.

```bash
# Try servers in order
data  -ro  server1:/data  server2:/data  server3:/data

# First available server is used
# Automatic failover if one fails
```

### Nested Automounts

**Scenario:** Organize mounts in hierarchies.

```bash
# Master map
/company /etc/auto.company

# /etc/auto.company
data    /etc/auto.company.data
projects /etc/auto.company.projects

# /etc/auto.company.data
current  -rw,soft  server:/data/current
archive  -ro,soft  server:/data/archive

# Result:
# /company/data/current
# /company/data/archive
# /company/projects/...
```

---

## Managing autofs

### Service Control

```bash
# Start autofs
systemctl start autofs

# Stop autofs (unmounts all autofs mounts)
systemctl stop autofs

# Reload configuration (safer, doesn't unmount)
systemctl reload autofs

# Restart (unmounts everything, reloads)
systemctl restart autofs

# Check status
systemctl status autofs

# Enable at boot
systemctl enable autofs
```

**Always use `reload` instead of `restart` when possible!**

### Checking Configuration

```bash
# Test configuration without starting
automount -f -v

# Run in debug mode (foreground)
systemctl stop autofs
automount -f -d

# View logs
journalctl -u autofs -f
tail -f /var/log/messages | grep automount
```

---

## Troubleshooting

### Problem: Mount Not Working

**Steps to diagnose:**

```bash
# Step 1: Check service
systemctl status autofs

# Step 2: Check configuration syntax
cat /etc/auto.master
cat /etc/auto.misc

# Step 3: Test access
ls /misc/share
cd /misc/share

# Step 4: Check logs
journalctl -u autofs --since "5 minutes ago"
dmesg | tail -20

# Step 5: Verify server
showmount -e nfs-server
ping nfs-server
```

**Common mistakes:**

```bash
# Wrong: Missing the colon for local device
cdrom   -fstype=iso9660,ro    /dev/cdrom

# Right: Add colon before local device
cdrom   -fstype=iso9660,ro    :/dev/cdrom

# Wrong: Full path in indirect map
/data/projects  -rw,soft  server:/projects

# Right: Just the key
projects  -rw,soft  server:/projects
```

### Problem: Permission Denied

**For NFS:**

```bash
# Check server exports
showmount -e server

# Verify client IP is allowed
exportfs -v

# Test manual mount
mount -t nfs server:/export /mnt/test
```

**For CIFS:**

```bash
# Check credentials file
cat /root/.smbcreds
ls -l /root/.smbcreds  # Should be 600

# Test with smbclient
smbclient //server/share -U username
```

### Problem: Mount Not Unmounting

**Check timeout:**

```bash
# View current timeout
grep timeout /etc/auto.master

# Increase timeout
/data /etc/auto.data --timeout=600
```

**Force unmount if needed:**

```bash
# Find what's using it
lsof /data/projects
fuser -m /data/projects

# Kill processes
fuser -k /data/projects

# Manual unmount
umount -l /data/projects
```

### Problem: Slow Performance

**Check network:**

```bash
# Test NFS performance
dd if=/dev/zero of=/autofs/mount/test bs=1M count=100

# Check mount options
mount | grep autofs
```

**Optimize options:**

```bash
# In map file, add performance options
data  -rw,soft,intr,rsize=32768,wsize=32768,noatime  server:/data
```

---

## Debug Mode

**When nothing works, use debug mode:**

```bash
# Stop service
systemctl stop autofs

# Run in foreground with debug
automount -f -d

# In another terminal, try to access mount
ls /autofs/path

# Watch debug output in first terminal
# Shows exactly what autofs is doing
```

**Example output:**

```
attempting to mount entry /misc/data
lookup_mount: lookup(file): looking up data
parse_mount: parse(sun): expanded entry: -rw,soft server:/data
mount_mount: mounting /misc/data from server:/data with options rw,soft
mounted /misc/data successfully
```

---

## Best Practices

**1. Always use `_netdev` equivalent options:**

```bash
# Don't rely on network at boot
# autofs handles network wait automatically
```

**2. Set reasonable timeouts:**

```bash
# Too short: Constant mounting/unmounting
# Too long: Resources held unnecessarily
# Good default: 300 seconds (5 minutes)
/data /etc/auto.data --timeout=300
```

**3. Use indirect maps when possible:**

```bash
# Cleaner, more organized
/shares /etc/auto.shares
```

**4. Secure credentials:**

```bash
chmod 600 /root/.smbcreds
chown root:root /root/.smbcreds
```

**5. Test before making permanent:**

```bash
# Test mount manually first
mount -t nfs server:/data /mnt/test

# Then add to autofs
```

**6. Use `reload` not `restart`:**

```bash
# After config changes
systemctl reload autofs
# Not: systemctl restart autofs
```

**7. Monitor autofs:**

```bash
# Regular checks
systemctl status autofs
journalctl -u autofs --since today
```

---

## Real-World Complete Example

**Scenario:** Company with multiple shares and user homes.

```bash
# === Step 1: Master Map ===
cat > /etc/auto.master << 'EOF'
/home     /etc/auto.home --timeout=600
/company  /etc/auto.company --timeout=300
/-        /etc/auto.direct
EOF

# === Step 2: Home Directories ===
cat > /etc/auto.home << 'EOF'
*    -rw,soft,intr,rsize=8192,wsize=8192    homeserver.company.local:/home/&
EOF

# === Step 3: Company Shares ===
cat > /etc/auto.company << 'EOF'
projects    -rw,soft,intr    proj-server:/projects
finance     -rw,soft,intr    fin-server:/finance
docs        -ro,soft         doc-server:/documentation
shared      -rw,soft,intr    file-server:/shared
EOF

# === Step 4: Direct Mounts ===
cat > /etc/auto.direct << 'EOF'
/data/warehouse    -rw,soft,intr    storage:/warehouse
/opt/tools         -ro,soft         tools:/export/tools
EOF

# === Step 5: Reload ===
systemctl reload autofs

# === Step 6: Test ===
# User homes
ls /home/john           # Auto-mounts homeserver:/home/john

# Company shares
ls /company/projects    # Auto-mounts proj-server:/projects
cd /company/finance     # Auto-mounts fin-server:/finance

# Direct mounts
cd /data/warehouse      # Auto-mounts storage:/warehouse

# === Step 7: Verify ===
mount | grep autofs
df -h | grep autofs

# === Step 8: Monitor ===
journalctl -u autofs -f
```

---

## Quick Reference

### Installation

```bash
# Install
dnf install autofs
systemctl enable --now autofs
```

### Configuration Files

```bash
# Master map (main config)
/etc/auto.master

# Map files (mount definitions)
/etc/auto.misc
/etc/auto.home
```

### Basic Indirect Map

```bash
# Master map
/mnt /etc/auto.mnt

# Map file
data  -rw,soft,intr  server:/export/data

# Result: /mnt/data
```

### Basic Direct Map

```bash
# Master map
/- /etc/auto.direct

# Map file
/data/warehouse  -rw,soft  server:/warehouse

# Result: /data/warehouse
```

### Service Management

```bash
systemctl reload autofs     # Apply changes (safe)
systemctl status autofs     # Check status
journalctl -u autofs -f     # View logs
automount -f -v             # Test config
```

### Testing

```bash
# Test by accessing
ls /mount/point

# Check if mounted
mount | grep autofs
df -h | grep autofs

# Debug mode
systemctl stop autofs
automount -f -d
```
