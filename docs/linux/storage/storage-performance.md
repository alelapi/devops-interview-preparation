# Monitor Storage Performance

## What is Storage Performance Monitoring?

Storage performance monitoring is about watching how your disks and filesystems are performing. Are they fast enough? Are they bottlenecking your system? Are they about to fail?

Think of it like checking your car's dashboard - you want to know if everything's running smoothly before problems occur.

### Why Monitor Storage?

**Prevent problems:**

- Catch failing disks before they fail completely
- Identify bottlenecks before users complain
- Plan capacity before running out of space

**Optimize performance:**

- Find slow I/O operations
- Identify what's causing disk activity
- Tune system for better performance

**Troubleshoot issues:**

- System is slow - is it the disk?
- Application timing out - disk bottleneck?
- High load - what's accessing the disk?

---

## Key Performance Metrics

### Understanding the Numbers

**IOPS (Input/Output Operations Per Second):**

- How many read/write operations per second
- Like "how many files can I access per second"
- Higher = better (especially for many small files)
- Example: Database servers need high IOPS

**Throughput (MB/s):**

- How much data transferred per second
- Like "how fast can I copy a large file"
- Higher = better (especially for large files)
- Example: Video editing needs high throughput

**Latency (milliseconds):**

- How long each operation takes
- Like "how long do I wait for disk response"
- Lower = better
- Good: <10ms, Acceptable: 10-20ms, Bad: >20ms

**Utilization (%):**

- How busy the disk is
- 100% means fully busy
- >80% sustained = potential bottleneck
- Example: 95% util = disk can't keep up

---

## iostat - I/O Statistics

### What is iostat?

iostat shows how busy your disks are, how much data they're reading/writing, and if there are performance issues.

**Think of it as:** A real-time dashboard for your disks.

### Installing iostat

```bash
# RHEL/CentOS/Rocky
dnf install sysstat
systemctl enable --now sysstat

# Debian/Ubuntu
apt install sysstat
systemctl enable --now sysstat
```

### Using iostat

**Examples:**

```bash
# Basic snapshot
iostat

# Extended statistics (most useful!)
iostat -x

# Human-readable sizes
iostat -xh

# Update every 2 seconds
iostat -x 2

# 10 samples, 2 seconds apart
iostat -x 2 10

# Show in megabytes
iostat -xm 2

# With timestamps
iostat -xt 2
```

**Real-world scenario - Check disk performance:**

```bash
# System feels slow, check disks
iostat -x 2

Device   r/s   w/s   rkB/s   wkB/s  await  %util
sda     10.5  45.2   512     2048   8.3    45.2
sdb    150.3  89.7  3072     4096   85.5   98.7
sdc      2.1   1.3    64       32   2.1     5.3
```

**Reading the output:**

- **sda:** Normal activity, 45% busy, 8ms wait - Good!
- **sdb:** Very busy (98.7%), high wait (85ms) - BOTTLENECK!
- **sdc:** Barely used, 5% busy - Fine

**Solution:** sdb is your problem. It's nearly 100% busy and operations are waiting 85ms.

### Key iostat Columns

```
Device:   Drive name (sda, sdb, nvme0n1)
r/s:      Reads per second
w/s:      Writes per second  
rkB/s:    Kilobytes read per second
wkB/s:    Kilobytes written per second
await:    Average wait time in milliseconds
%util:    How busy the drive is (percentage)
```

**What's good vs bad:**

```
await    Interpretation
<10ms    Excellent (SSD territory)
10-20ms  Good (normal HDD)
20-50ms  Slow (loaded system)
>50ms    Problem! (bottleneck)

%util    Interpretation
<50%     Plenty of capacity
50-80%   Moderate load
80-95%   Getting busy
>95%     Saturated (bottleneck!)
```

---

## iotop - I/O by Process

### What is iotop?

iotop shows which programs are using disk I/O. Like `top` but for disk activity.

**Think of it as:** "Who's hogging my disk?"

### Installing iotop

```bash
# RHEL/CentOS
dnf install iotop

# Debian/Ubuntu
apt install iotop
```

### Using iotop

**Examples:**

```bash
# Basic view
iotop

# Only show processes doing I/O (most useful!)
iotop -o

# Accumulated I/O (total since start)
iotop -oa

# With timestamps
iotop -oat

# Batch mode (for logging)
iotop -ob -n 10

# Monitor specific process
iotop -p 1234

# Monitor specific user
iotop -u mysql
```

**Interactive keys:**

- `o` - Toggle showing only active processes
- `a` - Toggle accumulated mode
- `r` - Reverse sort
- `q` - Quit

**Real-world scenario - System slow, find culprit:**

```bash
iotop -o

Total DISK READ: 85.3 MB/s | Total DISK WRITE: 120.5 MB/s
TID  USER     DISK READ  DISK WRITE  COMMAND
2341 mysql    65.3 MB/s   95.2 MB/s  mysqld
3422 root     15.2 MB/s   20.1 MB/s  tar
4551 www-data  4.8 MB/s    5.2 MB/s  apache2
```

**Reading the output:**

- MySQL is hammering the disk (65MB/s read, 95MB/s write)
- tar backup is also using disk
- Apache is minimal

**Solution:** MySQL query or backup causing high I/O.

---

## vmstat - System Statistics

### What is vmstat?

vmstat shows overall system performance including memory, swap, and I/O wait. Good for seeing if disk is causing system slowness.

**Think of it as:** Overall health monitor with disk focus.

### Using vmstat

**Examples:**

```bash
# Single snapshot
vmstat

# Update every 2 seconds
vmstat 2

# 10 updates
vmstat 2 10

# Show active/inactive memory
vmstat -a 2

# Disk statistics
vmstat -d

# Memory statistics
vmstat -s
```

**Real-world scenario - Is disk slowing system?**

```bash
vmstat 2

procs -----------memory---------- ---swap-- -----io---- --cpu----
 r  b   swpd   free   buff  cache   si   so    bi    bo   wa
 2  3  10240  2048  4096  8192     0    0   500  2000   35
 1  4  10240  2048  4096  8192     0    0   800  3500   45
 3  5  10240  2048  4096  8192     0    0  1200  5000   52
```

**Reading the output:**

- **r:** Processes waiting for CPU
- **b:** Processes blocked waiting for I/O (3-5 = high!)
- **si/so:** Swap in/out (0 = good, no swapping)
- **bi/bo:** Blocks in/out (high numbers = heavy I/O)
- **wa:** I/O wait (35-52% = PROBLEM!)

**Interpretation:**

When **wa** (I/O wait) is consistently above 20%, your system is waiting for disk operations. This is a bottleneck!

---

## df and du - Space Usage

### df - Disk Free

**What it does:** Shows how much space is used/available on filesystems.

**Think of it as:** "How full is my disk?"

**Examples:**

```bash
# Human-readable
df -h

# With filesystem types
df -hT

# Inode usage (number of files)
df -hi

# Specific filesystem
df -h /var
```

**Output example:**

```bash
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   35G   13G  73% /
/dev/sdb1       500G  475G   25G  95% /data
tmpfs           8.0G  1.0M  8.0G   1% /tmp
```

**Critical thresholds:**

- <80%: Healthy
- 80-90%: Monitor closely
- 90-95%: Take action soon
- >95%: Critical! Free space immediately

**Real-world scenario - Disk filling up:**

```bash
df -h /
# 95% full - need to find what's using space!

# Find large directories
du -sh /* | sort -rh | head -10
# Output:
# 30G  /var
# 10G  /usr
# 5G   /home

# Drill down
du -sh /var/* | sort -rh | head -5
# 25G  /var/log
# 3G   /var/cache
# 2G   /var/lib

# Found it! /var/log
ls -lh /var/log
# old-logs.tar.gz is 20GB!
```

### du - Disk Usage

**What it does:** Shows size of directories and files.

**Think of it as:** "What's taking up space?"

**Examples:**

```bash
# Current directory total
du -sh .

# Each subdirectory
du -sh *

# Top level only
du -h --max-depth=1 /var

# Sort by size
du -sh /var/* | sort -rh

# Top 10 largest
du -ah /home | sort -rh | head -10

# Exclude patterns
du -sh --exclude='*.log' /var
```

**Real-world scenario - Find space hogs:**

```bash
# Root is 95% full
df -h /

# Start at root
du -sh /* 2>/dev/null | sort -rh
# Output shows /var is huge

# Go deeper
du -sh /var/* | sort -rh
# /var/log is 50GB!

# Find biggest logs
du -sh /var/log/* | sort -rh | head -5
# application.log is 45GB!

# Clean up
gzip /var/log/application.log
# or delete old logs
```

---

## lsof and fuser - Find Open Files

### lsof - List Open Files

**What it does:** Shows which processes have which files open.

**Think of it as:** "Who's using this file/directory?"

**Examples:**

```bash
# All open files in directory
lsof +D /var/log

# Files opened by user
lsof -u mysql

# Files opened by process
lsof -p 1234

# What's using a mount point
lsof /mnt/data

# Network connections
lsof -i

# What's using port 80
lsof -i :80

# Find deleted but open files (wasting space!)
lsof | grep deleted
```

**Real-world scenario - Can't unmount:**

```bash
# Try to unmount
umount /data
# Error: device is busy

# Find what's using it
lsof +D /data
# Output:
# COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# bash    1234 john  cwd    DIR    8,1     4096   12 /data/work
# mysql   5678 mysql  4r    REG    8,1  5242880   45 /data/db/file.db

# Kill the processes or close their files
```

### fuser - Find Process Using File

**What it does:** Simpler than lsof, shows PIDs using a file.

**Think of it as:** Quick "who's using this?"

**Examples:**

```bash
# Show processes using file
fuser /var/log/syslog

# Verbose (show user and type)
fuser -v /mnt/data

# What's using the filesystem
fuser -m /mnt/data

# Kill all processes using it (dangerous!)
fuser -k /mnt/data

# Interactive kill (asks first)
fuser -ki /mnt/data
```

---

## smartctl - Disk Health

### What is smartctl?

smartctl reads SMART (Self-Monitoring, Analysis and Reporting Technology) data from drives. This predicts disk failures before they happen!

**Think of it as:** Your disk's health checkup.

### Installing smartctl

```bash
# RHEL/CentOS
dnf install smartmontools
systemctl enable --now smartd

# Debian/Ubuntu
apt install smartmontools
systemctl enable --now smartd
```

### Using smartctl

**Examples:**

```bash
# Health status (most important!)
smartctl -H /dev/sda

# All SMART data
smartctl -a /dev/sda

# Attributes only
smartctl -A /dev/sda

# Self-test log
smartctl -l selftest /dev/sda

# Error log
smartctl -l error /dev/sda

# Run short self-test
smartctl -t short /dev/sda

# Run long self-test (takes hours)
smartctl -t long /dev/sda

# For NVMe drives
smartctl -a /dev/nvme0
```

**Real-world scenario - Check disk health:**

```bash
# Monthly health check
smartctl -H /dev/sda

# Output:
# === START OF READ SMART DATA SECTION ===
# SMART overall-health self-assessment test result: PASSED

# Good! Now check details
smartctl -A /dev/sda
```

### Important SMART Attributes

```bash
ID  ATTRIBUTE_NAME          VALUE
5   Reallocated_Sector_Ct   100    ← Should be 0 or very low
10  Spin_Retry_Count         100    ← Should be 0
196 Reallocated_Event_Count  100    ← Should be 0
197 Current_Pending_Sector   100    ← Should be 0 (failing!)
198 Offline_Uncorrectable    100    ← Should be 0 (failing!)
```

**Warning signs (REPLACE DISK!):**

- Health status: FAILING
- Reallocated sectors increasing
- Current pending sectors > 0
- High error count
- Multiple failed self-tests

**Example - Failing disk:**

```bash
smartctl -A /dev/sdb

ID  ATTRIBUTE_NAME          VALUE
5   Reallocated_Sector_Ct    85    ← BAD! Was 100, now 85
197 Current_Pending_Sector    1    ← VERY BAD! Sectors failing
198 Offline_Uncorrectable     2    ← VERY BAD! Unreadable data

smartctl -H /dev/sdb
# Result: FAILING!

# ACTION REQUIRED:
# 1. Backup immediately!
# 2. Replace disk
# 3. Do NOT wait!
```

---

## Simple Performance Test

### dd - Basic Disk Speed Test

**What it does:** Tests raw disk read/write speed.

**Think of it as:** Simple benchmark.

**Examples:**

```bash
# Write test (1GB file)
dd if=/dev/zero of=/tmp/testfile bs=1M count=1000 oflag=direct

# Output:
# 1000+0 records in
# 1000+0 records out
# 1048576000 bytes (1.0 GB) copied, 5.2 s, 202 MB/s

# Read test
dd if=/tmp/testfile of=/dev/null bs=1M

# Cleanup
rm /tmp/testfile
```

**Interpreting results:**

- **HDD:** 100-200 MB/s typical
- **SATA SSD:** 500-600 MB/s
- **NVMe SSD:** 2000-7000 MB/s

---

## Troubleshooting Scenarios

### Scenario 1: System Very Slow

**Steps:**

```bash
# 1. Check I/O wait
vmstat 2 5
# If wa > 20%, disk is the problem

# 2. Find busy disk
iostat -x 2 5
# Look for %util > 90%

# 3. Find culprit process
iotop -o
# See what's hammering disk

# 4. Fix
# Kill process, optimize query, add cache, etc.
```

### Scenario 2: Disk Almost Full

**Steps:**

```bash
# 1. Confirm full
df -h

# 2. Find large directories
du -sh /* | sort -rh | head -10

# 3. Drill down
du -sh /var/* | sort -rh | head -5

# 4. Find specific files
find /var/log -type f -size +100M

# 5. Clean up
# Delete, archive, or move files
```

### Scenario 3: Cannot Unmount

**Steps:**

```bash
# 1. Find what's using it
lsof +D /mount/point
fuser -vm /mount/point

# 2. Kill processes
kill PID

# 3. If still busy, force
umount -l /mount/point
```

---

## Monitoring Best Practices

**1. Regular health checks:**

```bash
# Weekly disk health
smartctl -H /dev/sda

# Daily space check
df -h | grep -v tmpfs
```

**2. Set up alerts:**

```bash
# Alert when >90% full
df -h | awk '$5 > 90 {print $0}'

# Alert on SMART warnings
smartctl -H /dev/sda | grep -i fail
```

**3. Keep historical data:**

```bash
# Log iostat daily
iostat -x 60 1440 > /var/log/iostat-$(date +%Y%m%d).log
```

**4. Monitor trends:**

```bash
# Space usage over time
du -sh /var/log >> /var/log/space-usage.log
```

---

## Quick Reference

### Check Performance

```bash
iostat -x 2              # Disk busy?
iotop -o                 # Who's using disk?
vmstat 2                 # I/O wait high?
df -h                    # Space available?
```

### Find Space Hogs

```bash
du -sh /* | sort -rh     # Largest directories
du -sh * | sort -rh      # Current dir
find / -type f -size +1G # Files > 1GB
```

### Check Health

```bash
smartctl -H /dev/sda     # Overall health
smartctl -A /dev/sda     # Detailed attributes
dmesg | grep -i error    # System errors
```

### Troubleshoot Issues

```bash
lsof +D /path            # What's using path
fuser -vm /path          # PIDs using path
mount | grep /path       # Is it mounted?
```

### Speed Test

```bash
# Write speed
dd if=/dev/zero of=/tmp/test bs=1M count=1000 oflag=direct

# Read speed  
dd if=/tmp/test of=/dev/null bs=1M
```
