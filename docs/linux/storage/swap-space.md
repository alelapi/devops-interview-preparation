# Configure and Manage Swap Space

## What is Swap Space?

Swap is space on your hard drive that acts as extra (but slower) RAM. When your physical RAM fills up, Linux moves less-used data to swap to free up RAM for active programs.

Think of it like this:

- **RAM:** Your desk - fast and convenient for work you're doing now
- **Swap:** A filing cabinet - slower to access, but expands your working space

### When is Swap Used?

**Scenario 1: Running out of RAM**

You're running many programs and RAM fills up. Linux moves inactive pages to swap, freeing RAM for active processes.

**Scenario 2: Hibernation**

When you hibernate, Linux copies all RAM to swap, then powers off. On resume, it reads swap back into RAM.

**Scenario 3: Memory Management**

Even with free RAM, Linux might swap out rarely-used pages to have more free RAM for file caching (speeds up the system).

### Swap Size Recommendations

| Physical RAM | Swap Size (No Hibernation) | With Hibernation |
|--------------|---------------------------|------------------|
| < 2 GB       | 2x RAM                    | 3x RAM           |
| 2-8 GB       | = RAM                     | 2x RAM           |
| 8-64 GB      | 4-8 GB (or 0.5x RAM)     | 1.5x RAM         |
| > 64 GB      | 4-8 GB minimum            | Not practical    |

**Note:** These are guidelines. Your needs depend on your workload.

---

## Creating Swap Space

### Two Types of Swap

**1. Swap Partition:**

- Dedicated disk partition for swap
- Slightly faster
- Fixed size (harder to change)
- Traditional approach

**2. Swap File:**

- Regular file used as swap
- Easier to create/resize
- More flexible
- Modern approach

Both work equally well. Swap files are more convenient.

---

## Creating Swap Files

### fallocate - Quick File Creation

**What it does:** Instantly creates a file of specific size.

**Why use it:** Fastest way to create swap file.

**Example - Creating 2GB swap file:**

```bash
# Step 1: Create the file (instant!)
fallocate -l 2G /swapfile

# Step 2: Set correct permissions (CRITICAL for security!)
chmod 600 /swapfile

# Step 3: Format as swap
mkswap /swapfile

# Step 4: Enable it
swapon /swapfile

# Step 5: Verify
swapon --show
free -h

# Step 6: Make permanent
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

**Real-world scenario - VPS with no swap:**

```bash
# Many VPS providers don't include swap
# Check current swap
free -h
# Shows: Swap: 0B

# Create 4GB swap file
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Verify
free -h
# Now shows: Swap: 4.0Gi

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### dd - Alternative File Creation

**What it does:** Copies data to create file (slower but more compatible).

**Why use it:** Works on all systems, shows progress.

**Example:**

```bash
# Create 2GB swap file with progress
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress

# Then follow same steps as fallocate
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

---

## Creating Swap Partitions

### Using fdisk

**Real-world scenario - New disk for swap:**

```bash
# Step 1: Identify disk
lsblk
# /dev/sdb is your new disk

# Step 2: Create partition
fdisk /dev/sdb

# In fdisk:
# Press: n (new partition)
# Press: p (primary)
# Press: 1 (partition number)
# Press: Enter (default start)
# Press: +8G (8GB size)
# Press: t (change type)
# Press: 82 (Linux swap)
# Press: w (write changes)

# Step 3: Format as swap
mkswap /dev/sdb1

# Step 4: Enable
swapon /dev/sdb1

# Step 5: Make permanent
echo '/dev/sdb1 none swap sw 0 0' >> /etc/fstab
```

---

## Swap Commands

### mkswap - Format as Swap

**What it does:** Prepares a partition or file to be used as swap space.

**Why use it:** Required before using any space as swap.

**Examples:**

```bash
# Format partition
mkswap /dev/sdb1

# Format file
mkswap /swapfile

# With label (helpful for identification)
mkswap -L "swap1" /dev/sdb1
```

**Output example:**

```
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
no label, UUID=1234abcd-5678-...
```

### swapon - Enable Swap

**What it does:** Activates swap space so Linux can use it.

**Why use it:** Swap isn't usable until you enable it.

**Examples:**

```bash
# Enable specific swap
swapon /swapfile
swapon /dev/sdb1

# Enable all swap in /etc/fstab
swapon -a

# Enable with priority (higher number = used first)
swapon -p 10 /dev/sdb1
swapon -p 5 /swapfile

# Show what's active
swapon --show

# Verbose mode
swapon -v /swapfile
```

**Understanding priority:**

```bash
# Fast SSD swap - high priority (used first)
swapon -p 100 /dev/nvme0n1p3

# Slow HDD swap - low priority (used last)
swapon -p 10 /dev/sdb1
```

**Output of swapon --show:**

```
NAME      TYPE SIZE USED PRIO
/swapfile file   2G   0B   -2
/dev/sdb1 partition 4G 512M 5
```

### swapoff - Disable Swap

**What it does:** Deactivates swap space.

**Why use it:** Before removing swap or making changes.

**Examples:**

```bash
# Disable specific swap
swapoff /swapfile

# Disable all swap
swapoff -a

# Verbose
swapoff -v /swapfile
```

**Real-world scenario - Resizing swap file:**

```bash
# Current 2GB swap is too small, need 4GB

# Step 1: Disable swap
swapoff /swapfile

# Step 2: Delete old file
rm /swapfile

# Step 3: Create new 4GB file
fallocate -l 4G /swapfile
chmod 600 /swapfile

# Step 4: Format and enable
mkswap /swapfile
swapon /swapfile

# Step 5: Verify
swapon --show
```

### free - Memory and Swap Status

**What it does:** Shows RAM and swap usage.

**Why use it:** Quick check of memory situation.

**Examples:**

```bash
# Human-readable
free -h

# In megabytes
free -m

# In gigabytes  
free -g

# Continuous updates (every 2 seconds)
free -h -s 2

# With totals
free -ht

# Wide mode (more detailed)
free -hw
```

**Output example:**

```bash
free -h
              total        used        free      shared  buff/cache   available
Mem:           16Gi       8.0Gi       2.0Gi       100Mi       6.0Gi       7.5Gi
Swap:          4.0Gi       512Mi       3.5Gi
```

**What it means:**

- **total:** Total RAM/swap installed
- **used:** Currently in use by programs
- **free:** Completely unused
- **shared:** Used by tmpfs/shared memory
- **buff/cache:** Used for caching (can be freed if needed)
- **available:** How much can be used by programs (free + reclaimable cache)

### vmstat - Virtual Memory Statistics

**What it does:** Shows system activity including swap in/out.

**Why use it:** Monitor if system is swapping heavily (performance problem).

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

# Disk statistics
vmstat -d
```

**Output example:**

```bash
vmstat 2
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0  10240 204800 102400 512000    0    0    10    50  100  200  5  2 90  3  0
 0  1  10240 204800 102400 512000    5   10    20   100  150  250  8  4 85  3  0
```

**Critical columns:**

- **si (swap in):** KB/s reading from swap (from disk to RAM)
- **so (swap out):** KB/s writing to swap (from RAM to disk)
- **wa:** % CPU waiting for I/O

**What to watch for:**

```
si  so  ← Both zero: No swapping (good!)
0   0

si  so  ← Occasional swapping (normal)
5   10

si   so  ← Heavy swapping (performance problem!)
100  200
```

If you see high si/so consistently, you need more RAM!

---

## Persistent Swap Configuration

### /etc/fstab Entries

**Format:**

```
device/file  none  swap  options  0  0
```

**Examples:**

```bash
# Swap file
/swapfile none swap sw 0 0

# Swap partition
/dev/sdb1 none swap sw 0 0

# With priority
/dev/sdb1 none swap pri=10 0 0
/swapfile none swap pri=5 0 0

# Using UUID (more reliable)
UUID=1234abcd-... none swap sw 0 0

# Using label
LABEL=swap1 none swap sw 0 0
```

**Multiple swap spaces:**

```bash
# Fast SSD swap (priority 100)
/dev/nvme0n1p3 none swap pri=100 0 0

# Slower SSD swap (priority 50)
/dev/sda2 none swap pri=50 0 0

# Emergency HDD swap (priority 10)
/swapfile none swap pri=10 0 0
```

**Priority rules:**

- Higher number = used first
- Same priority = used in parallel (striped for performance)
- Default priority = -2

---

## Tuning Swap Behavior

### vm.swappiness - Control Swap Tendency

**What it is:** Kernel parameter controlling how aggressively Linux swaps.

**Why tune it:** Balance between RAM usage and swap usage.

**Value range:** 0-100

- **0:** Avoid swapping except emergency
- **1:** Minimum swapping (recommended for servers)
- **10:** Very little swapping (good for servers)
- **60:** Default (balanced)
- **100:** Swap aggressively

**Examples:**

```bash
# Check current value
cat /proc/sys/vm/swappiness
sysctl vm.swappiness

# Temporary change (until reboot)
sysctl -w vm.swappiness=10
echo 10 > /proc/sys/vm/swappiness

# Permanent change
echo "vm.swappiness=10" >> /etc/sysctl.d/99-swappiness.conf
sysctl -p /etc/sysctl.d/99-swappiness.conf

# Or edit /etc/sysctl.conf
vi /etc/sysctl.conf
# Add: vm.swappiness=10
sysctl -p
```

**Recommendations:**

```bash
# Desktop/Laptop (more responsive)
vm.swappiness=60

# Server with plenty of RAM
vm.swappiness=10

# Server with limited RAM
vm.swappiness=30

# Database server
vm.swappiness=1
```

**Real-world scenario - Web server optimization:**

```bash
# Server has 32GB RAM, barely uses swap
# Lower swappiness to keep more in RAM

# Check current setting
cat /proc/sys/vm/swappiness
# 60 (default)

# Set to minimal swapping
echo "vm.swappiness=10" >> /etc/sysctl.d/99-swappiness.conf
sysctl -p /etc/sysctl.d/99-swappiness.conf

# Monitor results
vmstat 2
# Watch si/so values (should be mostly 0)
```

---

## Swap on LVM

### Why Use LVM for Swap?

**Benefits:**

- Easy to resize
- Can be on multiple disks
- Snapshots possible (though not common for swap)

**Example - Create LVM swap:**

```bash
# Step 1: Create logical volume
lvcreate -L 8G -n lv_swap vg01

# Step 2: Format as swap
mkswap /dev/vg01/lv_swap

# Step 3: Enable
swapon /dev/vg01/lv_swap

# Step 4: Make permanent
echo '/dev/vg01/lv_swap none swap sw 0 0' >> /etc/fstab
```

**Resizing LVM swap:**

```bash
# Need more swap space

# Step 1: Disable swap
swapoff /dev/vg01/lv_swap

# Step 2: Extend logical volume
lvextend -L +4G /dev/vg01/lv_swap

# Step 3: Reformat (required for swap!)
mkswap /dev/vg01/lv_swap

# Step 4: Re-enable
swapon /dev/vg01/lv_swap

# Step 5: Verify
swapon --show
free -h
```

---

## Monitoring Swap Usage

### Check What's Using Swap

**Find swap usage by process:**

```bash
# Quick check
for file in /proc/*/status; do
    awk '/VmSwap|Name/{printf $2 " " $3}END{print ""}' $file
done | sort -k 2 -n -r | head

# Output shows:
# firefox 512000 (512MB)
# chrome 256000 (256MB)
# mysql 128000 (128MB)
```

**Detailed swap analysis:**

```bash
# For each process, show swap usage
for pid in $(ls /proc | grep -E '^[0-9]+$'); do
    if [ -f /proc/$pid/smaps ]; then
        swap=$(grep Swap /proc/$pid/smaps 2>/dev/null | awk '{sum+=$2} END {print sum}')
        if [ ! -z "$swap" ] && [ "$swap" -gt 0 ] 2>/dev/null; then
            name=$(cat /proc/$pid/comm 2>/dev/null)
            echo "$swap KB - $name (PID: $pid)"
        fi
    fi
done | sort -n -r | head -20
```

---

## Troubleshooting

### Problem: System Running Out of Memory

**Symptoms:** System very slow, heavy swapping.

**Solutions:**

```bash
# Check situation
free -h
vmstat 2 5

# If swap is full or nearly full:
# Option 1: Add more swap (temporary fix)
fallocate -l 4G /emergency-swap
chmod 600 /emergency-swap
mkswap /emergency-swap
swapon /emergency-swap

# Option 2: Find memory hogs
ps aux --sort=-%mem | head
top -o %MEM

# Option 3: Kill memory-hungry processes (carefully!)
pkill firefox
pkill chrome

# Long-term solution: Add more RAM!
```

### Problem: Heavy Swapping (System Slow)

**Symptoms:** High si/so in vmstat, system sluggish.

**Solutions:**

```bash
# Monitor swapping
vmstat 2

# If heavy swapping (si/so > 100):
# Solution 1: Lower swappiness
sysctl -w vm.swappiness=10

# Solution 2: Find what's swapped out
# (See "Check What's Using Swap" above)

# Solution 3: Clear swap and reload to RAM
# WARNING: Only if you have enough free RAM!
swapoff -a
swapon -a
```

### Problem: Cannot Enable Swap

**Symptoms:** swapon fails with error.

**Solutions:**

```bash
# Check if formatted as swap
file -s /swapfile
# Should show: swap file

# If not formatted:
mkswap /swapfile

# Check permissions (swap files must be 600)
ls -l /swapfile
chmod 600 /swapfile

# Check logs
dmesg | grep swap
journalctl | grep swap
```

### Problem: Swap Not Activating at Boot

**Symptoms:** After reboot, swap isn't active.

**Solutions:**

```bash
# Check fstab entry
cat /etc/fstab | grep swap

# Test manual activation
swapon -a

# Check for errors
systemctl status swap.target
systemctl list-units | grep swap

# Verify file/partition exists
ls -l /swapfile
```

---

## Best Practices

**1. Secure swap files:**

```bash
# Always set 600 permissions
chmod 600 /swapfile
```

**2. Use appropriate size:**

```bash
# Server with 16GB RAM
# 8GB swap is plenty
```

**3. Tune swappiness:**

```bash
# Servers: 10
# Desktops: 60 (default)
```

**4. Monitor regularly:**

```bash
# Check weekly
free -h
vmstat 2 5
```

**5. Multiple swap for performance:**

```bash
# Equal priority for striping
/dev/sda2 none swap pri=10 0 0
/dev/sdb2 none swap pri=10 0 0
```

---

## Quick Reference

### Creating Swap

```bash
# Swap file (recommended)
fallocate -l 4G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' >> /etc/fstab
```

### Managing Swap

```bash
swapon --show              # Show active swap
swapon -a                  # Enable all
swapoff -a                 # Disable all
free -h                    # Check usage
vmstat 2                   # Monitor activity
```

### Tuning

```bash
# Check swappiness
cat /proc/sys/vm/swappiness

# Set permanently
echo "vm.swappiness=10" >> /etc/sysctl.d/99-swap.conf
sysctl -p
```

### Monitoring

```bash
# Current status
free -h
swapon --show

# Watch for swapping
vmstat 2 10

# Find what's using swap
grep VmSwap /proc/*/status | grep -v "0 kB"
```
