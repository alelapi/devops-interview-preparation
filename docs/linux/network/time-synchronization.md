# Set and Synchronize System Time Using Time Servers

## Overview
This guide covers time synchronization, NTP (Network Time Protocol), Chrony, and time zone management in Linux systems.

---

## Time Concepts

- **System Time:** Current time maintained by the kernel
- **Hardware Clock (RTC):** Real-Time Clock, battery-powered clock in the hardware
- **UTC:** Coordinated Universal Time (standard reference)
- **Local Time:** Time adjusted for timezone
- **NTP:** Network Time Protocol for time synchronization
- **Stratum:** Distance from reference clock (lower is better, 0 is atomic clock)

---

## View Current Time

### `date` Command
```bash
# Show current date and time
date

# Show in UTC
date -u

# Show in specific format
date "+%Y-%m-%d %H:%M:%S"
date "+%A, %B %d, %Y"

# Show time in seconds since epoch (1970-01-01)
date +%s

# Show date from timestamp
date -d @1609459200
```

### `timedatectl` Command (systemd)
```bash
# Show time and date settings
timedatectl

# Output includes:
# - Local time
# - Universal time (UTC)
# - RTC time
# - Time zone
# - NTP enabled status
# - NTP synchronized status

# Show available timezones
timedatectl list-timezones

# Show time status in detail
timedatectl status
```

---

## Set System Time

### Manual Time Setting

#### Using `date`
```bash
# Set time manually (format: MMDDhhmmYYYY.ss)
date 123123592024.00  # December 31, 23:59:00, 2024

# Set from string
date -s "2024-12-31 23:59:00"
date -s "31 DEC 2024 23:59:00"

# Set date only
date --set="2024-12-31"

# Set time only
date --set="23:59:00"
```

#### Using `timedatectl`
```bash
# Set date and time
timedatectl set-time "2024-12-31 23:59:00"

# Set date only
timedatectl set-time "2024-12-31"

# Note: NTP must be disabled to set time manually
timedatectl set-ntp false
```

---

## Time Zone Management

### View Time Zone
```bash
# Using timedatectl
timedatectl

# Check timezone file
ls -l /etc/localtime

# View timezone name
cat /etc/timezone  # Debian/Ubuntu
```

### Set Time Zone

#### Using `timedatectl`
```bash
# List all available timezones
timedatectl list-timezones

# Filter timezones
timedatectl list-timezones | grep America
timedatectl list-timezones | grep Europe/

# Set timezone
timedatectl set-timezone America/New_York
timedatectl set-timezone Europe/London
timedatectl set-timezone Asia/Tokyo
timedatectl set-timezone UTC

# Verify
timedatectl
```

#### Manual Method
```bash
# Create symbolic link to timezone file
ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime

# Update /etc/timezone (Debian/Ubuntu)
echo "America/New_York" > /etc/timezone
```

---

## Hardware Clock (RTC)

### `hwclock` Command
```bash
# Show hardware clock time
hwclock --show
hwclock -r

# Show in UTC
hwclock --show --utc

# Show in local time
hwclock --show --localtime

# Set hardware clock from system time
hwclock --systohc

# Set system time from hardware clock
hwclock --hctosys

# Set hardware clock manually
hwclock --set --date="2024-12-31 23:59:00"
```

### Hardware Clock Mode
```bash
# Check if RTC is in UTC or local time
timedatectl | grep "RTC in local TZ"

# Set RTC to UTC (recommended)
timedatectl set-local-rtc 0

# Set RTC to local time (not recommended, can cause issues)
timedatectl set-local-rtc 1
```

---

## NTP - Network Time Protocol

### Check NTP Status
```bash
# Using timedatectl
timedatectl status
timedatectl timesync-status

# Show NTP service status
timedatectl show-timesync --all
```

### Enable/Disable NTP
```bash
# Enable NTP synchronization
timedatectl set-ntp true

# Disable NTP synchronization
timedatectl set-ntp false

# Check if enabled
timedatectl | grep "NTP service"
```

---

## Chrony (Modern NTP Implementation)

### Chrony Overview
Chrony is the modern replacement for ntpd, designed for:
- Better performance on systems that are intermittently connected
- Faster synchronization
- Better handling of network congestion
- Works well with virtual machines

### Installation
```bash
# RHEL/CentOS/Fedora
dnf install chrony

# Ubuntu/Debian
apt install chrony
```

### Chrony Service Management
```bash
# Start chronyd service
systemctl start chronyd

# Stop chronyd service
systemctl stop chronyd

# Restart chronyd service
systemctl restart chronyd

# Enable at boot
systemctl enable chronyd

# Check status
systemctl status chronyd

# Check if running
systemctl is-active chronyd
```

### Chrony Configuration File
Location: `/etc/chrony.conf` (RHEL) or `/etc/chrony/chrony.conf` (Ubuntu)

Example configuration:
```bash
# Use public NTP servers from pool.ntp.org
pool pool.ntp.org iburst
pool 0.pool.ntp.org iburst
pool 1.pool.ntp.org iburst

# Or use specific servers
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server time.google.com iburst

# Record rate of clock gain/loss
driftfile /var/lib/chrony/drift

# Allow system clock to be stepped in the first three updates
makestep 1.0 3

# Enable RTC synchronization
rtcsync

# Serve time to local network (optional)
allow 192.168.1.0/24

# Log directory
logdir /var/log/chrony
```

Configuration options explained:
- `pool`: Use pool of NTP servers
- `server`: Use specific NTP server
- `iburst`: Send burst of packets at startup for faster sync
- `makestep`: Allow clock to be stepped if offset is large
- `driftfile`: File to record clock drift rate
- `rtcsync`: Enable kernel RTC synchronization
- `allow`: Allow clients from this network

### `chronyc` Command (Chrony Client)

#### View Time Sources
```bash
# Show NTP sources
chronyc sources

# Show sources with verbose output
chronyc sources -v

# Columns explained:
# M: Mode (^ = server, = = peer)
# S: State (* = current sync source, + = acceptable, - = not acceptable)
# Stratum: Distance from reference clock
# Reach: Reachability (octal, 377 = all 8 recent attempts successful)
# LastRx: Time since last sample
# Poll: Polling interval

# Show detailed source information
chronyc sourcestats

# Show source information with verbose output
chronyc sourcestats -v
```

#### Tracking Information
```bash
# Show system clock performance
chronyc tracking

# Output includes:
# - Reference ID and stratum
# - System time offset
# - Root delay and dispersion
# - Update interval
# - Leap status
```

#### Manual Time Synchronization
```bash
# Force immediate synchronization
chronyc makestep

# Add new server temporarily
chronyc add server time.google.com

# Delete server
chronyc delete time.google.com
```

#### Activity and Statistics
```bash
# Show how many servers/peers are online
chronyc activity

# Show NTP authentication status
chronyc authdata

# Show client access
chronyc clients
```

#### Testing and Diagnostics
```bash
# Perform burst of measurements
chronyc burst 3/5

# Check server reachability
chronyc reachability

# Dump all measurements
chronyc ntpdata
```

---

## Legacy NTP Daemon (ntpd)

### Installation
```bash
# RHEL/CentOS/Fedora
dnf install ntp

# Ubuntu/Debian
apt install ntp
```

### Configuration
Location: `/etc/ntp.conf`

Example configuration:
```bash
# Use public servers
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst

# Drift file
driftfile /var/lib/ntp/drift

# Access restrictions
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1

# Allow local network to query
restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap
```

### Service Management
```bash
# Start service
systemctl start ntpd

# Enable at boot
systemctl enable ntpd

# Check status
systemctl status ntpd
```

### `ntpq` Command
```bash
# Show peers
ntpq -p

# Show more detailed peer information
ntpq -pn

# Interactive mode
ntpq
ntpq> peers
ntpq> associations
ntpq> quit
```

### `ntpdate` Command (deprecated)
```bash
# Synchronize time once (deprecated, use chronyc or ntpd)
ntpdate pool.ntp.org

# Note: Cannot run while ntpd is running
systemctl stop ntpd
ntpdate pool.ntp.org
systemctl start ntpd
```

---

## Systemd-timesyncd (Lightweight NTP)

### Overview
Simple SNTP client included with systemd, used when neither Chrony nor ntpd is installed.

### Configuration
Location: `/etc/systemd/timesyncd.conf`

```bash
[Time]
NTP=0.pool.ntp.org 1.pool.ntp.org
FallbackNTP=time.google.com
```

### Service Management
```bash
# Start service
systemctl start systemd-timesyncd

# Enable at boot
systemctl enable systemd-timesyncd

# Check status
systemctl status systemd-timesyncd

# View sync status
timedatectl timesync-status
```

---

## Public NTP Server Pools

### Common NTP Servers
```bash
# Generic pool
pool.ntp.org
0.pool.ntp.org
1.pool.ntp.org
2.pool.ntp.org
3.pool.ntp.org

# Regional pools
us.pool.ntp.org
europe.pool.ntp.org
asia.pool.ntp.org

# Vendor-specific pools
0.rhel.pool.ntp.org
0.ubuntu.pool.ntp.org
0.centos.pool.ntp.org

# Public servers
time.google.com
time.cloudflare.com
time.nist.gov
```

---

## Firewall Configuration for NTP

```bash
# Allow NTP traffic (UDP port 123)
firewall-cmd --permanent --add-service=ntp
firewall-cmd --reload

# Or manually with port
firewall-cmd --permanent --add-port=123/udp
firewall-cmd --reload

# For iptables
iptables -A INPUT -p udp --dport 123 -j ACCEPT
iptables -A OUTPUT -p udp --sport 123 -j ACCEPT
```

---

## Troubleshooting Time Synchronization

### Check Time Status
```bash
# Overall status
timedatectl status

# Chrony tracking
chronyc tracking
chronyc sources -v

# Check service
systemctl status chronyd
journalctl -u chronyd -f
```

### Common Issues and Solutions

#### 1. Time Not Synchronizing
```bash
# Check if NTP is enabled
timedatectl | grep "NTP service"

# Enable if disabled
timedatectl set-ntp true

# Restart chronyd
systemctl restart chronyd

# Check sources
chronyc sources -v

# Force synchronization
chronyc makestep
```

#### 2. Large Time Offset
```bash
# Check current offset
chronyc tracking

# If offset is very large, may need to step
chronyc makestep

# Or restart service
systemctl restart chronyd
```

#### 3. No Sources Available
```bash
# Check network connectivity
ping pool.ntp.org

# Check DNS resolution
nslookup pool.ntp.org

# Check firewall
firewall-cmd --list-all | grep ntp

# Test NTP port
nc -u pool.ntp.org 123
```

#### 4. Sources Not Reachable
```bash
# Check sources
chronyc sources

# Look at Reach column (should be 377 for good connectivity)
# If showing 0, check:
# - Firewall rules
# - Network connectivity
# - Server configuration

# Try different servers
chronyc add server time.google.com
```

### Log Files
```bash
# Chrony logs
journalctl -u chronyd
tail -f /var/log/chrony/*.log

# System logs
journalctl -xe

# Check for time-related errors
journalctl | grep -i "time\|clock\|ntp\|chrony"
```

---

## Best Practices

1. **Use Chrony** instead of ntpd for most modern systems
2. **Use pool servers** (pool.ntp.org) for redundancy
3. **Use iburst option** for faster initial synchronization
4. **Set RTC to UTC** to avoid timezone issues
5. **Monitor synchronization** regularly with chronyc tracking
6. **Allow firewall** access for NTP (UDP 123)
7. **Use regional pools** for better latency
8. **Keep chronyd running** continuously for best accuracy
9. **Check logs** regularly for synchronization issues
10. **Enable at boot** to ensure time is correct after restart

---

## Quick Reference Commands

### View Time
```bash
date                          # Current date/time
timedatectl                   # Full time status
timedatectl status            # Detailed status
hwclock --show                # Hardware clock
```

### Set Time
```bash
timedatectl set-time "YYYY-MM-DD HH:MM:SS"
timedatectl set-timezone America/New_York
timedatectl set-ntp true
```

### Chrony
```bash
systemctl start chronyd       # Start service
chronyc sources               # Show sources
chronyc tracking              # Show sync status
chronyc makestep              # Force sync
```

### Troubleshooting
```bash
timedatectl timesync-status   # Sync status
journalctl -u chronyd         # View logs
chronyc sources -v            # Detailed sources
chronyc activity              # Source activity
```

---

## Exam Tips

- Know how to use both `timedatectl` and `chronyc`
- Understand the difference between system time and hardware clock
- Be able to configure timezone correctly
- Know how to enable/disable NTP synchronization
- Understand chrony.conf configuration options
- Be able to troubleshoot synchronization issues
- Remember the `iburst` option for faster sync
- Know how to check synchronization status
- Understand the importance of UTC for RTC
- Practice manual time setting (with NTP disabled)
