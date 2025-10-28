# Configure and Manage Swap Space

## Overview
Swap space is a portion of storage used as virtual memory when RAM is full. It can be a dedicated partition or a file.

---

## Understanding Swap

### What is Swap?
- **Virtual Memory:** Extension of physical RAM
- **Paging:** Moving inactive pages to disk
- **Types:** Partition-based or file-based

### When is Swap Used?
- Physical RAM is full
- Swappiness parameter triggers swap usage
- System hibernation (suspend-to-disk)

### Swap Size Recommendations
| RAM Size | Swap Size (No Hibernation) | Swap Size (With Hibernation) |
|----------|---------------------------|------------------------------|
| < 2 GB   | 2x RAM                    | 3x RAM                       |
| 2-8 GB   | Equal to RAM              | 2x RAM                       |
| 8-64 GB  | 0.5x RAM (min 4GB)       | 1.5x RAM                     |
| > 64 GB  | 4GB minimum               | Hibernation not recommended  |

**Note:** These are guidelines. Actual needs depend on workload.

---

## Creating Swap Space

### Create Swap Partition

#### Using fdisk
```bash
fdisk /dev/sdb

# Commands in fdisk:
# n - new partition
# p - primary partition
# (accept defaults or specify size)
# t - change partition type
# 82 - Linux swap type
# w - write changes
```

#### Using parted
```bash
parted /dev/sdb
(parted) mklabel gpt
(parted) mkpart swap linux-swap 1GB 9GB
(parted) quit
```

**Use Cases:**
- Dedicated swap partition
- New system installation
- Adding swap to existing system

**Example Workflow:**
```bash
# Create partition
fdisk /dev/sdb
# In fdisk: n, p, 1, (default), +8G, t, 82, w

# Format as swap
mkswap /dev/sdb1

# Enable swap
swapon /dev/sdb1

# Verify
swapon --show
free -h
```

### Create Swap File

#### dd Method
```bash
dd if=/dev/zero of=/swapfile bs=1M count=2048
```

#### fallocate Method (Faster)
```bash
fallocate -l 2G /swapfile
```

**Common Options:**
- `-l` : Size (supports K, M, G, T)

**Use Cases:**
- Quick swap addition
- Systems without free partitions
- Temporary swap increase
- Cloud instances

**Complete Swap File Setup:**
```bash
# 1. Create swap file (2GB)
fallocate -l 2G /swapfile

# Or using dd
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress

# 2. Set correct permissions (CRITICAL for security!)
chmod 600 /swapfile

# 3. Format as swap
mkswap /swapfile

# 4. Enable swap
swapon /swapfile

# 5. Verify
swapon --show
free -h

# 6. Make persistent in /etc/fstab
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

---

## Swap Commands

### mkswap - Create Swap Area
```bash
mkswap [options] device|file
```
**Common Options:**
- `-L` : Set label
- `-U` : Set UUID
- `-c` : Check for bad blocks
- `-f` : Force creation

**Use Cases:**
- Initialize swap partition
- Format swap file
- Set swap label

**Examples:**
```bash
# Basic swap creation
mkswap /dev/sdb1

# With label
mkswap -L "swap1" /dev/sdb1

# For swap file
mkswap /swapfile

# Check for bad blocks
mkswap -c /dev/sdb1

# Force creation
mkswap -f /dev/sdb1
```

### swapon - Enable Swap
```bash
swapon [options] device|file
```
**Common Options:**
- `-a` : Enable all swap in /etc/fstab
- `-p` : Set priority
- `-s` or `--show` : Display swap usage
- `-v` : Verbose output

**Use Cases:**
- Activate swap space
- Enable swap with priority
- Display swap information

**Examples:**
```bash
# Enable specific swap
swapon /dev/sdb1
swapon /swapfile

# Enable all in /etc/fstab
swapon -a

# Enable with priority (higher = preferred)
swapon -p 10 /dev/sdb1
swapon -p 5 /swapfile

# Show swap usage
swapon --show
swapon -s

# Verbose
swapon -v /swapfile
```

### swapoff - Disable Swap
```bash
swapoff [options] device|file
```
**Common Options:**
- `-a` : Disable all swap
- `-v` : Verbose

**Use Cases:**
- Deactivate swap
- Before removing swap space
- System maintenance

**Examples:**
```bash
# Disable specific swap
swapoff /dev/sdb1
swapoff /swapfile

# Disable all swap
swapoff -a

# Verbose
swapoff -v /swapfile
```

### free - Memory and Swap Usage
```bash
free [options]
```
**Common Options:**
- `-h` : Human-readable
- `-m` : Show in MB
- `-g` : Show in GB
- `-k` : Show in KB
- `-s N` : Update every N seconds
- `-t` : Show total line
- `-w` : Wide mode (split cache)

**Use Cases:**
- Check memory usage
- Monitor swap usage
- System performance analysis

**Examples:**
```bash
# Basic output
free

# Human-readable
free -h

# In megabytes
free -m

# Continuous monitoring (every 2 seconds)
free -h -s 2

# With total
free -ht

# Wide mode
free -hw
```

### vmstat - Virtual Memory Statistics
```bash
vmstat [options] [delay] [count]
```
**Common Options:**
- `-a` : Active/inactive memory
- `-s` : Memory statistics
- `-d` : Disk statistics
- `-w` : Wide output

**Use Cases:**
- Monitor swap in/out
- Check system performance
- Identify memory pressure

**Examples:**
```bash
# Single snapshot
vmstat

# Update every 2 seconds
vmstat 2

# 10 updates, 2 seconds apart
vmstat 2 10

# Memory statistics
vmstat -s

# Active/inactive memory
vmstat -a

# Wide output
vmstat -w 2
```

**Key Columns:**
- `si` : Swap in (KB/s from disk)
- `so` : Swap out (KB/s to disk)
- `bi` : Blocks in (blocks/s)
- `bo` : Blocks out (blocks/s)

---

## Persistent Swap Configuration

### /etc/fstab - Permanent Swap
```bash
device_or_file  none  swap  options  0  0
```

**Examples:**
```bash
# Swap partition
/dev/sdb1 none swap sw 0 0

# Swap partition with priority
/dev/sdb1 none swap pri=10 0 0

# Swap file
/swapfile none swap sw 0 0

# Multiple swap spaces with priorities
/dev/sdb1 none swap pri=10 0 0
/swapfile none swap pri=5 0 0

# With label
LABEL=swap1 none swap sw 0 0

# With UUID
UUID=xxx-xxx-xxx none swap sw 0 0
```

### Swap Priority
- **Range:** -1 to 32767
- **Default:** -2
- **Higher number:** Higher priority (used first)
- **Equal priority:** Used in parallel (striped)

**Priority Use Cases:**
- Prefer faster SSD swap over HDD
- Balance I/O across devices
- Control swap usage order

**Example:**
```bash
# Fast SSD swap (high priority)
/dev/nvme0n1p2 none swap pri=100 0 0

# Slower HDD swap (low priority)
/dev/sdb1 none swap pri=10 0 0

# Swap file (lowest priority)
/swapfile none swap pri=1 0 0
```

---

## Tuning Swap Behavior

### vm.swappiness - Control Swap Tendency
```bash
# View current value
cat /proc/sys/vm/swappiness
sysctl vm.swappiness

# Temporary change
sysctl -w vm.swappiness=10
echo 10 > /proc/sys/vm/swappiness

# Permanent change (/etc/sysctl.conf or /etc/sysctl.d/99-swappiness.conf)
vm.swappiness=10
```

**Swappiness Values:**
- **0:** Disable swap (only use when absolutely necessary)
- **1:** Minimum swapping (kernel 3.5+)
- **10:** Recommended for servers
- **60:** Default value
- **100:** Aggressive swapping

**Use Cases:**
- **Desktops/Laptops:** 60 (default) or higher
- **Servers:** 1-10
- **High RAM systems:** 1-10
- **Low RAM systems:** 60+

**Examples:**
```bash
# Server optimization (minimal swap)
echo "vm.swappiness=10" >> /etc/sysctl.d/99-swappiness.conf
sysctl -p /etc/sysctl.d/99-swappiness.conf

# Desktop (more aggressive)
echo "vm.swappiness=60" >> /etc/sysctl.d/99-swappiness.conf
sysctl -p /etc/sysctl.d/99-swappiness.conf
```

### vm.vfs_cache_pressure - Cache Reclaim
```bash
# View current value
cat /proc/sys/vm/vfs_cache_pressure
sysctl vm.vfs_cache_pressure

# Temporary change
sysctl -w vm.vfs_cache_pressure=50

# Permanent
echo "vm.vfs_cache_pressure=50" >> /etc/sysctl.d/99-cache.conf
```

**Values:**
- **< 100:** Keep cache longer (good for file servers)
- **100:** Default (balanced)
- **> 100:** Reclaim cache more aggressively

---

## Managing Multiple Swap Spaces

### Check All Swap
```bash
# Show all active swap
swapon --show

# Or
cat /proc/swaps

# Memory details
free -h
```

### Remove Swap Space

#### Remove Swap Partition
```bash
# 1. Disable swap
swapoff /dev/sdb1

# 2. Remove from /etc/fstab
vi /etc/fstab
# Delete or comment out the line

# 3. (Optional) Delete partition
fdisk /dev/sdb
# Use 'd' to delete partition
```

#### Remove Swap File
```bash
# 1. Disable swap
swapoff /swapfile

# 2. Remove from /etc/fstab
vi /etc/fstab
# Delete or comment out the line

# 3. Delete file
rm /swapfile
```

### Resize Swap File
```bash
# 1. Disable current swap
swapoff /swapfile

# 2. Resize file
fallocate -l 4G /swapfile

# Or recreate
rm /swapfile
fallocate -l 4G /swapfile
chmod 600 /swapfile

# 3. Format
mkswap /swapfile

# 4. Enable
swapon /swapfile

# 5. Verify
swapon --show
```

---

## Swap on LVM

### Create Swap on LVM
```bash
# 1. Create logical volume
lvcreate -L 4G -n lv_swap vg01

# 2. Format as swap
mkswap /dev/vg01/lv_swap

# 3. Enable
swapon /dev/vg01/lv_swap

# 4. Add to /etc/fstab
echo '/dev/vg01/lv_swap none swap sw 0 0' >> /etc/fstab
```

### Extend LVM Swap
```bash
# 1. Disable swap
swapoff /dev/vg01/lv_swap

# 2. Extend LV
lvextend -L +2G /dev/vg01/lv_swap

# 3. Format
mkswap /dev/vg01/lv_swap

# 4. Enable
swapon /dev/vg01/lv_swap

# 5. Verify
swapon --show
free -h
```

---

## Swap Encryption

### Encrypted Swap with LUKS
```bash
# 1. Create encrypted device
cryptsetup luksFormat /dev/sdb1
cryptsetup luksOpen /dev/sdb1 swap_crypt

# 2. Format as swap
mkswap /dev/mapper/swap_crypt

# 3. Enable
swapon /dev/mapper/swap_crypt

# 4. Add to /etc/crypttab
echo 'swap_crypt /dev/sdb1 none' >> /etc/crypttab

# 5. Add to /etc/fstab
echo '/dev/mapper/swap_crypt none swap sw 0 0' >> /etc/fstab
```

### Random Key Encryption (No Hibernation)
```bash
# /etc/crypttab
swap /dev/sdb1 /dev/urandom swap,cipher=aes-xts-plain64,size=256

# /etc/fstab
/dev/mapper/swap none swap sw 0 0
```

---

## Monitoring Swap Usage

### Check Current Usage
```bash
# Overview
free -h
swapon --show

# Detailed
cat /proc/swaps
cat /proc/meminfo | grep -i swap
```

### Monitor Swap Activity
```bash
# Continuous monitoring
vmstat 2

# Watch swap in/out
watch -n 2 'cat /proc/swaps'

# Using htop
htop
# Press F2 -> Display options -> Show swap
```

### Find Processes Using Swap
```bash
# Show swap usage per process
for file in /proc/*/status; do 
    awk '/VmSwap|Name/{printf $2 " " $3}END{ print ""}' $file
done | sort -k 2 -n -r | head

# Or using smem (if installed)
smem -s swap

# Detailed view
for pid in $(ls /proc | grep -E '^[0-9]+$'); do
    if [ -f /proc/$pid/smaps ]; then
        swap=$(grep Swap /proc/$pid/smaps 2>/dev/null | awk '{sum+=$2} END {print sum}')
        if [ "$swap" -gt 0 ] 2>/dev/null; then
            name=$(cat /proc/$pid/comm 2>/dev/null)
            echo "$swap KB - $name (PID: $pid)"
        fi
    fi
done | sort -n -r | head
```

---

## Swap Troubleshooting

### Common Issues and Solutions

#### 1. System Running Out of Memory
```bash
# Check memory and swap
free -h
vmstat 1 5

# Identify memory hogs
ps aux --sort=-%mem | head
top -o %MEM

# Add more swap (temporary)
fallocate -l 2G /swapfile.emergency
chmod 600 /swapfile.emergency
mkswap /swapfile.emergency
swapon /swapfile.emergency
```

#### 2. Excessive Swapping (System Slow)
```bash
# Check swap activity
vmstat 2 10
# Look for high si/so values

# Reduce swappiness
sysctl -w vm.swappiness=10

# Identify processes using swap
# (see "Find Processes Using Swap" above)

# Add more RAM if possible
```

#### 3. Cannot Enable Swap
```bash
# Check if swap is formatted
file -s /dev/sdb1
blkid /dev/sdb1

# Reformat if needed
mkswap /dev/sdb1

# Check permissions (swap file)
ls -l /swapfile
# Should be 600
chmod 600 /swapfile

# Check for errors
dmesg | grep -i swap
journalctl | grep -i swap
```

#### 4. Swap Not Activating at Boot
```bash
# Verify /etc/fstab entry
cat /etc/fstab | grep swap

# Test mounting
swapon -a

# Check for errors
systemctl status swap.target
systemctl list-units | grep swap
```

#### 5. Swap Partition Not Recognized
```bash
# Check partition type
fdisk -l /dev/sdb
# Should show type 82 (Linux swap)

# Or with parted
parted /dev/sdb print

# Fix partition type if needed
fdisk /dev/sdb
# t, 82, w
```

---

## Best Practices

1. **Set Appropriate Size:** Follow RAM-based guidelines
2. **Secure Swap Files:** Always chmod 600
3. **Tune Swappiness:** Lower for servers (10), default for desktops (60)
4. **Monitor Usage:** Regular checks with vmstat and free
5. **Use SSD for Swap:** If possible, for better performance
6. **Multiple Swap Spaces:** Use priorities for optimization
7. **Consider Zswap:** Compressed swap in RAM (reduces disk I/O)
8. **Encrypt Swap:** For sensitive data
9. **Regular Testing:** Verify swap activates at boot
10. **Document Configuration:** Keep notes on custom settings

---

## Performance Tips

### 1. Use SSD for Swap
```bash
# Place swap on SSD for faster access
/dev/nvme0n1p2 none swap pri=100 0 0
```

### 2. Stripe Across Multiple Devices
```bash
# Equal priority = parallel usage
/dev/sdb1 none swap pri=10 0 0
/dev/sdc1 none swap pri=10 0 0
```

### 3. Optimize for Workload
```bash
# Low-memory server: aggressive swap
vm.swappiness=60
vm.vfs_cache_pressure=100

# High-memory server: minimal swap
vm.swappiness=10
vm.vfs_cache_pressure=50

# Database server: cache-focused
vm.swappiness=1
vm.vfs_cache_pressure=50
```

### 4. Use Zswap (Compressed Swap in RAM)
```bash
# Enable zswap (usually enabled by default in modern kernels)
echo 1 > /sys/module/zswap/parameters/enabled

# Check status
cat /sys/module/zswap/parameters/enabled

# Configure compressor
echo lz4 > /sys/module/zswap/parameters/compressor
```

---

## Quick Reference

### Create Swap
```bash
# Swap file
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

# Swap partition
mkswap /dev/sdb1
swapon /dev/sdb1
```

### Manage Swap
```bash
swapon --show                    # Show active swap
swapon -a                        # Enable all (fstab)
swapoff -a                       # Disable all
free -h                          # Memory/swap usage
vmstat 2                         # Monitor activity
```

### Tune Swap
```bash
# Reduce swapping
sysctl -w vm.swappiness=10

# Make permanent
echo "vm.swappiness=10" >> /etc/sysctl.d/99-swap.conf
```

### /etc/fstab Entry
```bash
/swapfile none swap sw 0 0
/dev/sdb1 none swap pri=10 0 0
```
