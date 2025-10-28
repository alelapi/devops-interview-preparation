# Monitor Storage Performance

## Overview
This guide covers tools and techniques for monitoring disk I/O, filesystem performance, and identifying storage bottlenecks.

---

## Key Performance Metrics

### Understanding Storage Metrics
- **IOPS:** Input/Output Operations Per Second
- **Throughput:** Data transfer rate (MB/s)
- **Latency:** Time to complete I/O operation
- **Queue Depth:** Number of pending I/O requests
- **Utilization:** Percentage of time device is busy
- **Wait Time:** Time waiting for I/O completion

### Storage Performance Indicators
- **High Utilization (>80%):** Potential bottleneck
- **High Wait Times:** Slow storage or heavy load
- **High Queue Depth:** More requests than device can handle
- **Low Throughput:** Network or disk limitations

---

## iostat - I/O Statistics

### Overview
Reports CPU and device I/O statistics. Part of sysstat package.

### Installation
```bash
# RHEL/CentOS/Rocky
dnf install sysstat

# Debian/Ubuntu
apt install sysstat

# Enable sysstat
systemctl enable --now sysstat
```

### iostat - Basic Usage
```bash
iostat [options] [interval] [count]
```

**Common Options:**
- `-x` : Extended statistics (detailed)
- `-d` : Device statistics only
- `-c` : CPU statistics only
- `-k` : Display in KB/s
- `-m` : Display in MB/s
- `-h` : Human-readable format
- `-p` : Show partitions
- `-t` : Include timestamp
- `-N` : Display by device mapper name

**Use Cases:**
- Identify I/O bottlenecks
- Monitor disk utilization
- Track throughput and IOPS
- Performance baseline

**Examples:**
```bash
# Basic output (one snapshot)
iostat

# Extended statistics
iostat -x

# Human-readable with extended stats
iostat -xh

# Monitor every 2 seconds
iostat -x 2

# 10 samples, 2 seconds apart
iostat -x 2 10

# In megabytes
iostat -xm 2

# Show partitions
iostat -xp sda

# Device mapper names
iostat -xN

# With timestamps
iostat -xt 2

# CPU and disk together
iostat -xc 2
```

### Understanding iostat Output

#### Key Columns Explained
```bash
Device:   Device name
rrqm/s:   Read requests merged per second
wrqm/s:   Write requests merged per second
r/s:      Read requests per second (IOPS read)
w/s:      Write requests per second (IOPS write)
rkB/s:    Kilobytes read per second (throughput read)
wkB/s:    Kilobytes written per second (throughput write)
avgrq-sz: Average request size in sectors
avgqu-sz: Average queue length
await:    Average wait time (ms) - includes queue + service
r_await:  Average read wait time (ms)
w_await:  Average write wait time (ms)
svctm:    Average service time (deprecated)
%util:    Device utilization percentage
```

**Interpreting Results:**
- **%util > 80%:** Device is busy, possible bottleneck
- **await > 10ms:** Slow response times (depends on device)
- **avgqu-sz > 1:** Queue building up
- **High rrqm/s, wrqm/s:** Good, requests being merged efficiently

**Example Output:**
```bash
Device   r/s   w/s   rkB/s   wkB/s  await  %util
sda     45.2  78.3   1024    3072   12.5   85.2
```
**Analysis:** High utilization (85.2%), moderate wait time, possible bottleneck

---

## iotop - I/O by Process

### Overview
Shows I/O usage by processes, similar to top but for disk I/O.

### Installation
```bash
# RHEL/CentOS/Rocky
dnf install iotop

# Debian/Ubuntu
apt install iotop
```

### iotop - Usage
```bash
iotop [options]
```

**Common Options:**
- `-o` : Only show processes doing I/O
- `-b` : Batch mode (non-interactive)
- `-n NUM` : Number of iterations
- `-d SEC` : Delay between updates
- `-p PID` : Monitor specific process
- `-u USER` : Monitor specific user
- `-P` : Show processes instead of threads
- `-a` : Accumulated I/O
- `-k` : Use kilobytes
- `-t` : Add timestamp

**Interactive Keys:**
- `r` : Reverse sort order
- `o` : Toggle showing only active processes
- `p` : Toggle between processes/threads
- `a` : Toggle accumulated mode
- `q` : Quit

**Use Cases:**
- Identify which processes cause high I/O
- Find I/O-intensive applications
- Debug performance issues
- Monitor specific users or processes

**Examples:**
```bash
# Basic usage
iotop

# Only show processes doing I/O
iotop -o

# Batch mode (for scripts/logs)
iotop -b -n 3

# Monitor specific process
iotop -p 1234

# Monitor specific user
iotop -u www-data

# With timestamps and accumulated I/O
iotop -oat

# Show processes, not threads
iotop -oP

# Output to file
iotop -b -n 10 -d 2 > iotop.log
```

### Understanding iotop Output
```bash
Total DISK READ: 5.2 MB/s | Total DISK WRITE: 12.8 MB/s
TID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN    IO    COMMAND
1234 be/4  mysql    2.5 MB/s   8.2 MB/s    0.00%    45%   mysqld
5678 be/4  root     1.1 MB/s   2.3 MB/s    0.00%    18%   updatedb
```

**Columns:**
- **TID:** Thread/Process ID
- **PRIO:** I/O priority
- **DISK READ/WRITE:** Current I/O rates
- **SWAPIN:** Percentage of time swapping
- **IO:** Percentage of time waiting for I/O
- **COMMAND:** Process name

---

## vmstat - Virtual Memory Statistics

### Overview
Reports virtual memory, process, CPU, and I/O statistics.

### vmstat - Usage
```bash
vmstat [options] [delay] [count]
```

**Common Options:**
- `-a` : Active/inactive memory
- `-d` : Disk statistics
- `-D` : Disk summary
- `-s` : Memory statistics
- `-p partition` : Partition statistics
- `-S` : Units (k/K/m/M)

**Use Cases:**
- Monitor system performance
- Check swap activity
- View I/O wait
- Overall system health

**Examples:**
```bash
# Basic output (since boot)
vmstat

# Update every 2 seconds
vmstat 2

# 10 samples, 2 seconds apart
vmstat 2 10

# With active/inactive memory
vmstat -a 2

# Disk statistics
vmstat -d

# Disk summary
vmstat -D

# Memory statistics
vmstat -s

# Partition statistics
vmstat -p sda1

# In megabytes
vmstat -S M 2
```

### Understanding vmstat Output

#### Key Columns
```bash
procs:
  r: Processes waiting for CPU
  b: Processes in uninterruptible sleep (blocked I/O)

memory:
  swpd: Virtual memory used
  free: Free memory
  buff: Buffer memory
  cache: Cache memory

swap:
  si: Swap in (from disk)
  so: Swap out (to disk)

io:
  bi: Blocks in (read from disk)
  bo: Blocks out (written to disk)

system:
  in: Interrupts per second
  cs: Context switches per second

cpu:
  us: User CPU time
  sy: System CPU time
  id: Idle CPU time
  wa: I/O wait time
  st: Stolen time (virtualization)
```

**Interpreting Results:**
- **High wa (>20%):** System waiting for I/O (disk bottleneck)
- **High b:** Many processes blocked on I/O
- **High si/so:** Heavy swapping (need more RAM)
- **High bi/bo:** Heavy disk I/O

**Example:**
```bash
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  3  12000  2000   4000  8000    0    0   500  2000  100  200 10  5 60 25  0
```
**Analysis:** 3 processes blocked, 25% I/O wait, 500 blocks read/2000 written per second

---

## sar - System Activity Reporter

### Overview
Collects, reports, and saves system activity information. Part of sysstat package.

### sar - Usage
```bash
sar [options] [interval] [count]
```

**Common Options:**
- `-b` : I/O and transfer rate statistics
- `-d` : Block device statistics
- `-n DEV` : Network statistics
- `-r` : Memory statistics
- `-S` : Swap statistics
- `-u` : CPU statistics
- `-p` : Pretty-print device names
- `-f file` : Read from log file

**Use Cases:**
- Historical performance analysis
- Generate performance reports
- Identify trends
- Capacity planning

**Examples:**
```bash
# Current I/O statistics
sar -b

# Block device statistics
sar -d

# Every 2 seconds, 5 times
sar -d 2 5

# From log file (historical data)
sar -d -f /var/log/sa/sa28

# All block devices
sar -dp

# Memory usage
sar -r 2 5

# Swap usage
sar -S 2 5

# CPU with I/O wait
sar -u 2 5

# Network statistics
sar -n DEV 2 5
```

### Understanding sar Output

#### I/O Statistics (sar -b)
```bash
tps:      Transfers per second (IOPS)
rtps:     Read transfers per second
wtps:     Write transfers per second
bread/s:  Blocks read per second
bwrtn/s:  Blocks written per second
```

#### Block Device Statistics (sar -d)
```bash
DEV:      Device name
tps:      Transfers per second
rd_sec/s: Sectors read per second
wr_sec/s: Sectors written per second
avgrq-sz: Average request size
avgqu-sz: Average queue size
await:    Average wait time
svctm:    Service time (deprecated)
%util:    Utilization percentage
```

---

## df - Filesystem Usage

### df - Disk Free Space
```bash
df [options] [filesystem]
```

**Common Options:**
- `-h` : Human-readable sizes
- `-T` : Show filesystem type
- `-i` : Show inode information
- `-a` : Include all filesystems
- `-t type` : Limit to filesystem type
- `-x type` : Exclude filesystem type
- `--total` : Show total

**Use Cases:**
- Check available space
- Monitor filesystem usage
- Identify full filesystems
- Capacity planning

**Examples:**
```bash
# Human-readable output
df -h

# With filesystem type
df -hT

# Inode usage
df -ih

# Specific filesystem
df -h /var

# Only ext4 filesystems
df -hT -t ext4

# Exclude tmpfs
df -hT -x tmpfs

# With totals
df -h --total

# All filesystems including pseudo
df -ha
```

### Understanding df Output
```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       50G   30G   18G  63%  /
/dev/sdb1      200G  150G   40G  79%  /data
```

**Critical Thresholds:**
- **>80% used:** Monitor closely
- **>90% used:** Take action soon
- **>95% used:** Critical, free space immediately

---

## du - Disk Usage

### du - Estimate File Space Usage
```bash
du [options] [directory]
```

**Common Options:**
- `-h` : Human-readable sizes
- `-s` : Summary (total only)
- `-c` : Grand total
- `-a` : All files, not just directories
- `--max-depth=N` : Limit depth
- `--exclude=pattern` : Exclude files
- `-x` : Stay on same filesystem
- `--time` : Show modification time

**Use Cases:**
- Find large directories
- Disk space analysis
- Identify space hogs
- Cleanup planning

**Examples:**
```bash
# Current directory size
du -sh

# All items in directory
du -h --max-depth=1 /var

# Sorted by size
du -h /var | sort -h

# Top 10 largest directories
du -h /var | sort -rh | head -10

# With grand total
du -sch /var/*

# Show all files
du -ah /var/log

# Exclude certain patterns
du -h --exclude='*.log' /var

# Stay on same filesystem
du -hx /

# With modification times
du -h --time /backup
```

### Find Large Files
```bash
# Files larger than 100MB
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Top 20 largest files
find / -type f -exec du -h {} \; 2>/dev/null | sort -rh | head -20

# Files in /var larger than 50MB
find /var -type f -size +50M -exec du -h {} \; | sort -rh
```

---

## lsof - List Open Files

### lsof - Usage
```bash
lsof [options] [path]
```

**Common Options:**
- `+D dir` : All open files under directory
- `-u user` : Files opened by user
- `-p PID` : Files opened by process
- `-c command` : Files opened by command
- `-i` : Network connections
- `-t` : Terse output (PIDs only)

**Use Cases:**
- Find which process uses a file
- Troubleshoot "device busy" errors
- Check open file handles
- Network connection analysis

**Examples:**
```bash
# Files open in directory
lsof +D /var/log

# Files opened by user
lsof -u www-data

# Files opened by process
lsof -p 1234

# Files opened by command
lsof -c mysql

# What's using a mount point
lsof /mnt/data

# Deleted files still open (wasting space)
lsof | grep deleted

# Open network connections
lsof -i

# Specific port
lsof -i :80

# PIDs using a file
lsof -t /var/log/syslog
```

---

## fuser - Identify Processes Using Files

### fuser - Usage
```bash
fuser [options] file|directory
```

**Common Options:**
- `-m` : All files on filesystem
- `-v` : Verbose output
- `-k` : Kill processes using file
- `-u` : Show username
- `-i` : Interactive (ask before kill)

**Use Cases:**
- Find processes preventing unmount
- Identify file usage
- Kill processes using specific files

**Examples:**
```bash
# Show processes using file
fuser /var/log/syslog

# Verbose with user info
fuser -vu /mnt/data

# What's using the filesystem
fuser -vm /mnt/data

# Kill processes using file (dangerous!)
fuser -k /var/log/app.log

# Interactive kill
fuser -ki /mnt/data
```

---

## smartctl - SMART Disk Monitoring

### Overview
Monitors disk health using SMART (Self-Monitoring, Analysis and Reporting Technology).

### Installation
```bash
# RHEL/CentOS/Rocky
dnf install smartmontools

# Debian/Ubuntu
apt install smartmontools

# Enable daemon
systemctl enable --now smartd
```

### smartctl - Usage
```bash
smartctl [options] device
```

**Common Options:**
- `-i` : Show device information
- `-a` : All SMART information
- `-H` : Health status
- `-A` : Attributes
- `-l error` : Error log
- `-l selftest` : Self-test log
- `-t short` : Run short self-test
- `-t long` : Run long self-test

**Use Cases:**
- Check disk health
- Predict disk failures
- Run disk tests
- Monitor disk wear

**Examples:**
```bash
# Device information
smartctl -i /dev/sda

# Health status
smartctl -H /dev/sda

# All SMART data
smartctl -a /dev/sda

# Attributes only
smartctl -A /dev/sda

# Error log
smartctl -l error /dev/sda

# Self-test log
smartctl -l selftest /dev/sda

# Run short test
smartctl -t short /dev/sda

# Run long test
smartctl -t long /dev/sda

# For NVMe devices
smartctl -a /dev/nvme0
```

### Key SMART Attributes
```bash
ID  ATTRIBUTE_NAME          VALUE
  5 Reallocated_Sector_Ct   Important: Should be 0
  9 Power_On_Hours          Usage time
 10 Spin_Retry_Count        Should be low
196 Reallocated_Event_Count Important: Should be 0
197 Current_Pending_Sector  Important: Should be 0
198 Offline_Uncorrectable   Important: Should be 0
```

**Warning Signs:**
- Increasing reallocated sectors
- Current pending sectors
- High read error rate
- Health status: FAILING

---

## blktrace - Block Layer I/O Tracing

### Overview
Detailed block layer I/O tracing for advanced debugging.

### Installation
```bash
# RHEL/CentOS/Rocky
dnf install blktrace

# Debian/Ubuntu
apt install blktrace
```

### blktrace - Usage
```bash
# Start tracing
blktrace -d /dev/sda -o trace

# In another terminal, run workload
# Stop with Ctrl+C

# Parse trace
blkparse -i trace

# Generate statistics
btt -i trace.blktrace.0
```

**Use Cases:**
- Deep I/O analysis
- Performance debugging
- Understanding I/O patterns

---

## ioping - I/O Latency Measurement

### Overview
Measure disk I/O latency in real-time, similar to ping for disks.

### Installation
```bash
# RHEL/CentOS/Rocky
dnf install ioping

# Debian/Ubuntu
apt install ioping
```

### ioping - Usage
```bash
ioping [options] directory|device
```

**Common Options:**
- `-c count` : Number of requests
- `-i interval` : Interval between requests
- `-s size` : Request size
- `-S wsize` : Write size for temp files
- `-R` : Use random seek
- `-W` : Use writes instead of reads

**Examples:**
```bash
# Test latency
ioping /var

# 10 requests
ioping -c 10 /var

# Random seeks
ioping -R /var

# Write test
ioping -W /tmp

# Specific size
ioping -s 4k /var
```

---

## hdparm - Hard Disk Parameters

### hdparm - Usage
```bash
hdparm [options] device
```

**Common Options:**
- `-I` : Detailed information
- `-i` : Identification info
- `-t` : Read timing test
- `-T` : Cache read timing test
- `-g` : Display geometry
- `-W` : Set/get write cache

**Use Cases:**
- Test read performance
- Check disk capabilities
- Configure disk parameters

**Examples:**
```bash
# Disk information
hdparm -I /dev/sda

# Read test
hdparm -t /dev/sda

# Cache test
hdparm -T /dev/sda

# Both tests
hdparm -tT /dev/sda

# Drive geometry
hdparm -g /dev/sda

# Check write cache
hdparm -W /dev/sda
```

---

## Performance Monitoring Scripts

### Continuous Monitoring Script
```bash
#!/bin/bash
# monitor_io.sh - Monitor I/O performance

LOG_FILE="/var/log/io_monitor.log"
INTERVAL=5

echo "=== I/O Monitoring Started at $(date) ===" >> "$LOG_FILE"

while true; do
    echo "--- $(date) ---" >> "$LOG_FILE"
    
    # iostat
    iostat -x 1 1 >> "$LOG_FILE"
    
    # Top I/O processes
    echo "Top I/O Processes:" >> "$LOG_FILE"
    iotop -b -n 1 -o >> "$LOG_FILE" 2>/dev/null
    
    # Disk usage
    echo "Disk Usage:" >> "$LOG_FILE"
    df -h >> "$LOG_FILE"
    
    sleep "$INTERVAL"
done
```

### Alert Script
```bash
#!/bin/bash
# disk_alert.sh - Alert on high disk usage

THRESHOLD=90
EMAIL="admin@example.com"

df -h | tail -n +2 | while read line; do
    usage=$(echo "$line" | awk '{print $5}' | sed 's/%//')
    partition=$(echo "$line" | awk '{print $6}')
    
    if [ "$usage" -ge "$THRESHOLD" ]; then
        echo "WARNING: $partition is ${usage}% full" | \
            mail -s "Disk Usage Alert" "$EMAIL"
    fi
done
```

---

## Troubleshooting Performance Issues

### High I/O Wait
```bash
# Identify cause
iostat -x 2 5
iotop -o

# Check which processes
ps aux --sort=-%cpu | head
top -o %CPU

# Review logs
dmesg | grep -i error
journalctl -p err -n 50
```

### Slow Filesystem
```bash
# Check mount options
mount | grep /slow/filesystem

# Test performance
dd if=/dev/zero of=/slow/filesystem/testfile bs=1M count=1000
rm /slow/filesystem/testfile

# Check for errors
dmesg | grep -i "I/O error"
journalctl -f
```

### Full Filesystem
```bash
# Find large files
find /filesystem -type f -size +100M -exec ls -lh {} \;

# Find large directories
du -h /filesystem | sort -rh | head -20

# Find old files
find /filesystem -type f -mtime +90

# Check for deleted but open files
lsof | grep deleted
```

---

## Best Practices

1. **Regular Monitoring:** Use tools like sar for continuous monitoring
2. **Set Baselines:** Know normal performance patterns
3. **Monitor Trends:** Watch for gradual degradation
4. **Check SMART Data:** Regular disk health checks
5. **Tune I/O Scheduler:** Choose appropriate scheduler (noop, deadline, cfq, bfq)
6. **Use Monitoring Tools:** Implement Prometheus, Grafana, Nagios
7. **Alert on Thresholds:** Automated alerts for issues
8. **Regular Cleanup:** Remove old files and logs
9. **Capacity Planning:** Monitor growth trends
10. **Document Baselines:** Keep records of normal performance

---

## Quick Reference

### Quick Checks
```bash
# Current I/O status
iostat -x 1 3

# Top I/O processes
iotop -o

# Filesystem usage
df -hT

# Large directories
du -h --max-depth=1 / | sort -rh | head -10

# Disk health
smartctl -H /dev/sda

# What's using filesystem
lsof +D /mount/point
```

### Performance Testing
```bash
# Read test
dd if=/dev/sda of=/dev/null bs=1M count=1000

# Write test
dd if=/dev/zero of=/tmp/testfile bs=1M count=1000

# Disk benchmark
hdparm -tT /dev/sda

# Latency test
ioping -c 20 /var
```

### Common Flags
```bash
iostat -x        # Extended stats
iotop -o         # Only active
vmstat 2         # 2-second intervals
sar -d           # Block devices
df -hT           # Human-readable with types
du -sh           # Summary
```
