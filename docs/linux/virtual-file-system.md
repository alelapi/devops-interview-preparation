# Manage and Configure the Virtual File System

## Overview
The Virtual File System (VFS) is an abstraction layer that provides a unified interface for accessing different filesystem types. Understanding VFS and system directories is crucial for Linux administration.

---

## Filesystem Hierarchy Standard (FHS)

### Key System Directories

#### /bin - Essential User Binaries
```bash
ls /bin
```
**Contents:** Essential command binaries (ls, cp, mv, cat, etc.)
**Use Cases:**
- Commands needed in single-user mode
- Basic system utilities

#### /boot - Boot Loader Files
```bash
ls /boot
```
**Contents:** Kernel, initramfs, bootloader configuration
**Key Files:**
- `vmlinuz-*` : Linux kernel
- `initramfs-*` or `initrd-*` : Initial RAM filesystem
- `grub/` : GRUB bootloader configuration

#### /dev - Device Files
```bash
ls -l /dev
```
**Contents:** Device files (block and character devices)
**Important Devices:**
- `/dev/sda`, `/dev/sdb` : SCSI/SATA disks
- `/dev/nvme0n1` : NVMe devices
- `/dev/null` : Null device
- `/dev/zero` : Zero device
- `/dev/random`, `/dev/urandom` : Random data
- `/dev/tty*` : Terminal devices

**Use Cases:**
- Direct device access
- System diagnostics

#### /etc - System Configuration
```bash
ls /etc
```
**Contents:** System-wide configuration files
**Key Files:**
- `/etc/fstab` : Filesystem mount table
- `/etc/hosts` : Static hostname resolution
- `/etc/passwd` : User account information
- `/etc/group` : Group information
- `/etc/shadow` : Secure user passwords
- `/etc/ssh/` : SSH configuration
- `/etc/network/` : Network configuration

#### /home - User Home Directories
```bash
ls /home
```
**Contents:** Personal directories for users

#### /lib, /lib64 - Shared Libraries
```bash
ls /lib
```
**Contents:** Essential shared libraries and kernel modules

#### /media - Removable Media
```bash
ls /media
```
**Contents:** Mount points for removable devices (USB, CD/DVD)

#### /mnt - Temporary Mount Points
```bash
ls /mnt
```
**Contents:** Temporary mount points for administrators

#### /opt - Optional Software
```bash
ls /opt
```
**Contents:** Third-party application packages

#### /proc - Process Information
```bash
ls /proc
cat /proc/cpuinfo
cat /proc/meminfo
```
**Contents:** Virtual filesystem with kernel and process information
**Important Files:**
- `/proc/cpuinfo` : CPU information
- `/proc/meminfo` : Memory information
- `/proc/[pid]/` : Process-specific information
- `/proc/sys/` : Kernel parameters (sysctl)
- `/proc/mounts` : Currently mounted filesystems
- `/proc/partitions` : Partition information

**Use Cases:**
- System monitoring
- Process management
- Runtime kernel parameter tuning

#### /root - Root User Home
```bash
ls /root
```
**Contents:** Home directory for root user

#### /run - Runtime Data
```bash
ls /run
```
**Contents:** Runtime variable data (since boot)
**Key Directories:**
- `/run/systemd/` : systemd runtime data
- `/run/lock/` : Lock files
- `/run/user/` : User session data

#### /sbin - System Binaries
```bash
ls /sbin
```
**Contents:** Essential system administration binaries
**Examples:** fdisk, mkfs, mount, iptables

#### /srv - Service Data
```bash
ls /srv
```
**Contents:** Data for services (web, FTP)

#### /sys - System Information
```bash
ls /sys
```
**Contents:** Virtual filesystem for device and kernel information
**Key Directories:**
- `/sys/block/` : Block devices
- `/sys/class/` : Device classes
- `/sys/devices/` : Physical devices

**Use Cases:**
- Hardware information
- Device management
- Kernel module parameters

#### /tmp - Temporary Files
```bash
ls /tmp
```
**Contents:** Temporary files (cleared on reboot)
**Permissions:** Usually 1777 (sticky bit)

#### /usr - User Programs
```bash
ls /usr
```
**Contents:** User applications and utilities
**Subdirectories:**
- `/usr/bin/` : User commands
- `/usr/sbin/` : System administration commands
- `/usr/lib/` : Libraries
- `/usr/local/` : Locally installed software
- `/usr/share/` : Shared data

#### /var - Variable Data
```bash
ls /var
```
**Contents:** Variable files that change during operation
**Key Directories:**
- `/var/log/` : Log files
- `/var/spool/` : Spool directories (mail, print)
- `/var/tmp/` : Temporary files (preserved across reboots)
- `/var/cache/` : Application cache
- `/var/lib/` : State information

---

## File Types in Linux

### Identify File Types
```bash
ls -l
file filename
stat filename
```

### File Type Indicators (ls -l)
- `-` : Regular file
- `d` : Directory
- `l` : Symbolic link
- `b` : Block device
- `c` : Character device
- `p` : Named pipe (FIFO)
- `s` : Socket

**Example:**
```bash
ls -l /dev/sda
brw-rw---- 1 root disk 8, 0 Oct 28 10:00 /dev/sda
# b = block device
```

---

## Working with Directories

### mkdir - Create Directories
```bash
mkdir directory_name
```
**Common Options:**
- `-p` : Create parent directories as needed
- `-m` : Set permissions
- `-v` : Verbose output

**Use Cases:**
- Create directory structure
- Set permissions during creation

**Examples:**
```bash
# Create single directory
mkdir /opt/myapp

# Create nested directories
mkdir -p /opt/myapp/config/prod

# Create with specific permissions
mkdir -m 755 /opt/myapp

# Create multiple directories
mkdir dir1 dir2 dir3

# Create directory structure
mkdir -p project/{src,bin,doc,test}
```

### rmdir - Remove Empty Directories
```bash
rmdir directory_name
```
**Common Options:**
- `-p` : Remove parent directories if empty
- `--ignore-fail-on-non-empty` : Don't error on non-empty

**Use Cases:**
- Remove empty directories
- Clean up directory structure

**Examples:**
```bash
# Remove empty directory
rmdir /tmp/testdir

# Remove nested empty directories
rmdir -p /tmp/test/nested/dir
```

### rm - Remove Files and Directories
```bash
rm filename
```
**Common Options:**
- `-r` or `-R` : Recursive (for directories)
- `-f` : Force (no prompt)
- `-i` : Interactive (prompt)
- `-v` : Verbose

**Use Cases:**
- Delete files and directories
- Force removal of protected files

**Examples:**
```bash
# Remove file
rm file.txt

# Remove directory and contents
rm -rf /tmp/olddata

# Interactive removal
rm -i *.txt

# Remove with confirmation for each file
rm -rI directory/
```

### pwd - Print Working Directory
```bash
pwd
```
**Common Options:**
- `-P` : Print physical path (resolve symlinks)
- `-L` : Print logical path (with symlinks)

**Use Cases:**
- Show current directory
- Verify location in scripts

### cd - Change Directory
```bash
cd /path/to/directory
```
**Special Paths:**
- `cd` or `cd ~` : Go to home directory
- `cd -` : Go to previous directory
- `cd ..` : Go up one level
- `cd ../..` : Go up two levels

**Examples:**
```bash
cd /var/log
cd ~
cd -
cd ../../etc
```

---

## Listing and Finding Files

### ls - List Directory Contents
```bash
ls [options] [path]
```
**Common Options:**
- `-l` : Long format (permissions, owner, size, date)
- `-a` : Show hidden files (starting with .)
- `-h` : Human-readable sizes
- `-R` : Recursive listing
- `-t` : Sort by modification time
- `-r` : Reverse order
- `-S` : Sort by size
- `-i` : Show inode numbers
- `--color` : Colorize output

**Use Cases:**
- View file details
- Find recently modified files
- Check permissions and ownership

**Examples:**
```bash
# Detailed listing with human-readable sizes
ls -lh

# Show all files including hidden
ls -la

# Sort by modification time, newest first
ls -lt

# Show inode numbers
ls -li

# Recursive listing
ls -lR /etc

# Sort by size
ls -lhS

# Combined: all files, long format, human sizes
ls -lah
```

### find - Search for Files
```bash
find [path] [options] [expression]
```
**Common Options:**
- `-name` : Search by name (case-sensitive)
- `-iname` : Search by name (case-insensitive)
- `-type` : File type (f=file, d=directory, l=link)
- `-size` : File size
- `-mtime` : Modified time in days
- `-atime` : Access time in days
- `-perm` : Permissions
- `-user` : Owner
- `-group` : Group
- `-exec` : Execute command on results
- `-delete` : Delete found files

**Use Cases:**
- Locate files by name, size, or date
- Find and execute commands on files
- System cleanup

**Examples:**
```bash
# Find files by name
find /home -name "*.txt"

# Find files case-insensitive
find /var -iname "*.log"

# Find directories
find /opt -type d -name "config"

# Find files larger than 100MB
find /var/log -type f -size +100M

# Find files modified in last 7 days
find /home -type f -mtime -7

# Find files modified more than 30 days ago
find /tmp -type f -mtime +30

# Find and delete old files
find /tmp -type f -mtime +7 -delete

# Find files with specific permissions
find /var/www -type f -perm 0777

# Find and execute command
find /var/log -name "*.log" -exec gzip {} \;

# Find empty files
find /tmp -type f -empty

# Find files owned by user
find /home -user john

# Find large files and list by size
find / -type f -size +1G -exec ls -lh {} \; 2>/dev/null

# Find and change permissions
find /var/www -type d -exec chmod 755 {} \;
find /var/www -type f -exec chmod 644 {} \;
```

### locate - Quick File Search
```bash
locate filename
```
**Common Options:**
- `-i` : Case-insensitive
- `-c` : Count matches
- `-l` : Limit results
- `-r` : Use regex

**Use Cases:**
- Fast file location (uses database)
- Find files across system

**Examples:**
```bash
# Find files
locate nginx.conf

# Case-insensitive
locate -i README

# Update database first
updatedb
locate newfile.txt
```

### which - Locate Command Binary
```bash
which command
```
**Use Cases:**
- Find executable location
- Verify command in PATH

**Examples:**
```bash
which python3
which -a python   # Show all matches
```

### whereis - Locate Binary, Source, and Manual
```bash
whereis command
```
**Common Options:**
- `-b` : Binary only
- `-m` : Manual only
- `-s` : Source only

**Examples:**
```bash
whereis ls
whereis -b nginx
```

---

## File Permissions and Attributes

### chmod - Change File Permissions
```bash
chmod [options] mode file
```
**Common Options:**
- `-R` : Recursive
- `-v` : Verbose

**Numeric Mode:**
- 4 = read (r)
- 2 = write (w)
- 1 = execute (x)

**Symbolic Mode:**
- u = user, g = group, o = others, a = all
- + = add, - = remove, = = set exact

**Use Cases:**
- Set file permissions
- Control access to files and directories

**Examples:**
```bash
# Numeric mode
chmod 755 script.sh      # rwxr-xr-x
chmod 644 file.txt       # rw-r--r--
chmod 600 private.key    # rw-------
chmod 777 shared/        # rwxrwxrwx (not recommended)

# Symbolic mode
chmod u+x script.sh      # Add execute for user
chmod go-w file.txt      # Remove write for group and others
chmod a+r document.pdf   # Add read for all
chmod u=rwx,go=rx dir/   # Set exact permissions

# Recursive
chmod -R 755 /var/www/html
```

### chown - Change File Owner
```bash
chown [options] user[:group] file
```
**Common Options:**
- `-R` : Recursive
- `-v` : Verbose
- `--reference=` : Use reference file

**Use Cases:**
- Change file ownership
- Fix permission issues

**Examples:**
```bash
# Change owner
chown john file.txt

# Change owner and group
chown john:developers file.txt

# Change only group (or use chgrp)
chown :developers file.txt

# Recursive
chown -R www-data:www-data /var/www

# Use reference file
chown --reference=ref.txt file.txt
```

### chgrp - Change Group Ownership
```bash
chgrp [options] group file
```
**Common Options:**
- `-R` : Recursive
- `-v` : Verbose

**Examples:**
```bash
chgrp developers project/
chgrp -R www-data /var/www/site
```

### umask - Set Default Permissions
```bash
umask [mask]
```
**Use Cases:**
- Set default permissions for new files
- Security policy enforcement

**Examples:**
```bash
# View current umask
umask

# Set umask (022 = 755 for dirs, 644 for files)
umask 022

# Set umask (027 = 750 for dirs, 640 for files)
umask 027

# In /etc/profile or ~/.bashrc
umask 022
```

**Permission Calculation:**
- Directories: 777 - umask
- Files: 666 - umask

---

## File Attributes

### lsattr - List File Attributes
```bash
lsattr [file]
```
**Common Options:**
- `-a` : Show all files
- `-d` : List directory attributes
- `-R` : Recursive

**Use Cases:**
- View special file attributes
- Check immutable flags

**Examples:**
```bash
lsattr file.txt
lsattr -a /etc
```

### chattr - Change File Attributes
```bash
chattr [operator][attribute] file
```
**Common Attributes:**
- `i` : Immutable (cannot be modified, deleted, or renamed)
- `a` : Append only
- `A` : No atime updates
- `d` : No dump
- `s` : Secure deletion

**Operators:**
- `+` : Add attribute
- `-` : Remove attribute
- `=` : Set exact attributes

**Use Cases:**
- Protect files from deletion
- Prevent accidental modifications
- Optimize performance

**Examples:**
```bash
# Make file immutable
chattr +i /etc/important.conf

# Remove immutable flag
chattr -i /etc/important.conf

# Append only (for logs)
chattr +a /var/log/custom.log

# No atime updates (performance)
chattr +A /var/log/access.log
```

---

## Links

### ln - Create Links
```bash
ln [options] target link_name
```
**Common Options:**
- `-s` : Create symbolic (soft) link
- `-f` : Force (overwrite existing)
- `-v` : Verbose
- `-i` : Interactive

**Hard Links vs Symbolic Links:**
- **Hard Link:** Same inode, same data blocks
- **Symbolic Link:** Separate inode, contains path to target

**Use Cases:**
- Create shortcuts
- Maintain backward compatibility
- Link configuration files

**Examples:**
```bash
# Create symbolic link
ln -s /usr/local/bin/python3.9 /usr/local/bin/python

# Create hard link
ln /original/file.txt /backup/file.txt

# Force overwrite
ln -sf /new/target /existing/link

# Link directory
ln -s /opt/application/current /opt/app

# Multiple links
ln -s /source/file /link1 /link2
```

---

## /proc Filesystem

### Important /proc Files

#### System Information
```bash
# CPU information
cat /proc/cpuinfo

# Memory information
cat /proc/meminfo

# Kernel version
cat /proc/version

# System uptime
cat /proc/uptime

# Load average
cat /proc/loadavg

# Mounted filesystems
cat /proc/mounts
```

#### Process Information
```bash
# Process details
ls /proc/[pid]/
cat /proc/[pid]/status
cat /proc/[pid]/cmdline
cat /proc/[pid]/environ

# Process limits
cat /proc/[pid]/limits

# Open files
ls -l /proc/[pid]/fd/
```

#### Network Information
```bash
# Network interfaces
cat /proc/net/dev

# Routing table
cat /proc/net/route

# ARP table
cat /proc/net/arp
```

---

## /sys Filesystem

### Important /sys Directories

#### Block Devices
```bash
# List block devices
ls /sys/block/

# Device information
cat /sys/block/sda/size
cat /sys/block/sda/ro
```

#### Network Devices
```bash
# Network interfaces
ls /sys/class/net/

# Interface status
cat /sys/class/net/eth0/operstate
cat /sys/class/net/eth0/address
```

#### Power Management
```bash
# CPU frequency
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```

---

## Useful Commands for VFS Management

### df - Disk Space Usage
```bash
df -h
```
**Common Options:**
- `-h` : Human-readable
- `-T` : Show filesystem type
- `-i` : Show inode usage

**Examples:**
```bash
df -h
df -Th
df -i
```

### du - Directory Space Usage
```bash
du -sh directory
```
**Common Options:**
- `-s` : Summary
- `-h` : Human-readable
- `-c` : Total
- `--max-depth=N` : Depth limit

**Examples:**
```bash
du -sh /var/log
du -h --max-depth=1 /home
du -sch /var/log/* | sort -h
```

### tree - Display Directory Tree
```bash
tree [directory]
```
**Common Options:**
- `-L` : Max depth
- `-a` : Show hidden files
- `-d` : Directories only
- `-h` : Human-readable sizes

**Examples:**
```bash
tree -L 2 /etc
tree -dL 1 /
```

---

## Best Practices

1. **Never modify /proc or /sys directly** unless you know what you're doing
2. **Use /mnt for temporary mounts**, /media for removable media
3. **Keep /tmp cleaned up** regularly
4. **Monitor /var/log** for disk space issues
5. **Use symbolic links** instead of hard links for directories
6. **Set proper umask** in user profiles
7. **Document custom mounts** in /etc/fstab
8. **Backup configuration files** in /etc before changes
