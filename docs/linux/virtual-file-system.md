# Manage and Configure the Virtual File System

## What is the Virtual File System?

The Virtual File System (VFS) is Linux's way of providing a uniform interface to access different types of filesystems. It's like a translator that lets applications work with files the same way, whether they're on ext4, XFS, NFS, or any other filesystem.

Think of VFS as the middleman between your applications and the actual storage. When you run `cat file.txt`, you don't need to know if that file is on a local disk, a network share, or even in memory - VFS handles all the complexity.

---

## Understanding the Linux Directory Structure

### Why This Matters

Unlike Windows with its C:, D:, E: drives, Linux has ONE unified directory tree. Everything starts from `/` (root), and all storage devices, network shares, and even system information appear as directories within this tree.

### The Root Directory - /

Everything in Linux starts here. This is not the home directory of the root user (that's `/root`), but the very top of the entire filesystem hierarchy.

```bash
# View the root directory
ls /
```

**Output:**
```
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

---

## Critical System Directories

### /bin - Essential User Commands

**What it is:** Contains essential command-line programs that all users need and that must be available even in single-user mode (emergency recovery).

**What's inside:**

- Basic commands like `ls`, `cp`, `mv`, `cat`, `mkdir`
- Shell programs like `bash`, `sh`
- Basic utilities you need to fix a broken system

**Example:**

```bash
# See what's in /bin
ls /bin

# Check where a command lives
which ls
# Output: /bin/ls
```

**Why it matters:** If these files get deleted or corrupted, your system becomes very difficult to use or repair.

### /boot - Boot Files

**What it is:** Everything needed to start your Linux system.

**What's inside:**

- The Linux kernel (vmlinuz)
- Initial RAM disk (initramfs or initrd)
- Bootloader configuration (GRUB)

**Example:**

```bash
# View boot files
ls -lh /boot

# Typical contents:
# vmlinuz-5.15.0-58-generic    (Linux kernel)
# initrd.img-5.15.0-58-generic (Initial RAM disk)
# grub/                        (Bootloader)
```

**Why it matters:** Your computer can't start Linux without these files. Never delete anything here unless you know exactly what you're doing!

**Real-world scenario:**
If you're dual-booting Windows and Linux and Windows update breaks GRUB, you'll need to access `/boot/grub/` to fix the bootloader.

### /dev - Device Files

**What it is:** Special files that represent hardware devices. In Linux, "everything is a file," including hardware.

**What's inside:**

- Disk drives: `/dev/sda`, `/dev/sdb`, `/dev/nvme0n1`
- Partitions: `/dev/sda1`, `/dev/sda2`
- Terminals: `/dev/tty1`, `/dev/pts/0`
- Null device: `/dev/null` (the "black hole")
- Random data: `/dev/random`, `/dev/urandom`

**Examples:**

```bash
# List all disk devices
ls -l /dev/sd*

# List NVMe devices
ls -l /dev/nvme*

# View information about a disk
fdisk -l /dev/sda

# The "black hole" - discard output
echo "This disappears" > /dev/null

# Generate random data
head -c 16 /dev/urandom | base64
```

**Real-world scenario:**
When you plug in a USB drive, it appears as `/dev/sdb` or similar. You can then mount it to access its contents.

### /etc - Configuration Files

**What it is:** System-wide configuration files. Nearly every program stores its settings here.

**What's inside:**

- `/etc/fstab` - Filesystem mount configuration
- `/etc/passwd` - User account information
- `/etc/group` - Group information
- `/etc/hosts` - Static hostname to IP mappings
- `/etc/hostname` - System hostname
- `/etc/ssh/sshd_config` - SSH server configuration
- `/etc/network/` - Network configuration

**Examples:**

```bash
# View mount configuration
cat /etc/fstab

# Check system hostname
cat /etc/hostname

# View user accounts
cat /etc/passwd

# List all configuration files
ls /etc
```

**Why it matters:** This is where you configure most system behavior. Always backup files before editing them!

**Real-world scenario:**
You want a USB drive to mount automatically on boot. You add an entry to `/etc/fstab`:
```
UUID=1234-5678 /mnt/usb vfat defaults 0 0
```

### /home - User Home Directories

**What it is:** Personal directories for regular users. Each user gets their own space.

**What's inside:**

- `/home/john/` - John's personal files
- `/home/jane/` - Jane's personal files
- Each user can only write to their own directory (by default)

**Examples:**

```bash
# Go to your home directory
cd ~
# or just
cd

# See whose home directories exist
ls /home

# Check your current user
whoami

# Check your home directory
echo $HOME
```

**Why separate /home?** Many people put `/home` on a separate partition or disk. This way, you can reinstall the operating system without losing your personal files.

### /root - Root User's Home

**What it is:** The home directory for the root (administrator) user.

**Why separate?** The root user is special. Their home directory is in `/root` (not `/home/root`) to ensure it's available even if `/home` fails to mount.

**Example:**

```bash
# Switch to root
sudo su -

# Check location
pwd
# Output: /root

# Go back to regular user
exit
```

### /tmp - Temporary Files

**What it is:** A place for programs to store temporary data. Gets cleaned out regularly (usually on reboot).

**What's inside:**

- Temporary files created by applications
- Lock files
- Session data

**Examples:**

```bash
# Create a temp file
echo "test" > /tmp/mytest.txt

# List temp files
ls /tmp

# Many systems clean /tmp on reboot
# So don't store anything important here!
```

**Why it matters:** Perfect for testing or temporary work. After reboot, it's clean.

**Real-world scenario:**
You're testing a script that creates files. Use `/tmp` so you don't clutter your home directory, and it auto-cleans:
```bash
./test-script.sh > /tmp/output.log
# After testing, reboot cleans it up automatically
```

### /var - Variable Data

**What it is:** Data that changes frequently during system operation.

**What's inside:**

- `/var/log/` - Log files
- `/var/spool/` - Print and mail queues
- `/var/www/` - Web server files (common location)
- `/var/lib/` - State information for applications
- `/var/cache/` - Cached data

**Examples:**

```bash
# View system logs
ls /var/log

# Check recent log entries
tail /var/log/syslog
tail /var/log/messages

# Web server files (if Apache installed)
ls /var/www/html

# Check disk usage (can grow large!)
du -sh /var/log
```

**Why it matters:** `/var/log` can fill up your disk! Regular monitoring is essential.

**Real-world scenario:**
Your root partition is full. Investigation shows:
```bash
df -h /
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        20G   19G     0 100% /

du -sh /var/log
# 15G    /var/log

# Old logs are filling the disk!
# Solution: Clean old logs
find /var/log -name "*.log" -mtime +30 -delete
```

### /usr - User Programs and Data

**What it is:** Unix System Resources. Contains most user applications and utilities.

**What's inside:**

- `/usr/bin/` - Most command-line programs
- `/usr/sbin/` - System administration programs
- `/usr/lib/` - Libraries for programs
- `/usr/local/` - Locally compiled/installed software
- `/usr/share/` - Shared data (documentation, icons, etc.)

**Examples:**

```bash
# Most commands you use are here
ls /usr/bin

# Where is Python?
which python3
# Output: /usr/bin/python3

# Where is Apache?
which apache2
# Output: /usr/sbin/apache2

# Locally installed software
ls /usr/local/bin
```

**Why /usr/local?** System package managers install to `/usr`, but software you compile yourself goes to `/usr/local`. This keeps them separate and organized.

---

## Special Virtual Filesystems

### /proc - Process and System Information

**What it is:** Not a real filesystem on disk! It's a window into the kernel's view of the system. Everything here is generated in real-time.

**What's inside:**

- System information (CPU, memory, etc.)
- Process information (one directory per running process)
- Kernel parameters you can read and change

**Examples:**

```bash
# CPU information
cat /proc/cpuinfo

# Memory information
cat /proc/meminfo

# System load
cat /proc/loadavg

# Currently mounted filesystems
cat /proc/mounts

# Network connections
cat /proc/net/tcp

# Information about process 1234
ls /proc/1234/
cat /proc/1234/status
cat /proc/1234/cmdline

# All running processes have a directory
ls /proc/ | grep -E '^[0-9]+$'
```

**Real-world scenario - Checking memory:**
```bash
# How much RAM do I have?
grep MemTotal /proc/meminfo
# MemTotal:       16384000 kB

# How much is free?
grep MemAvailable /proc/meminfo
# MemAvailable:   8192000 kB
```

**Real-world scenario - Finding what's listening on a port:**
```bash
# What's listening on port 80?
cat /proc/net/tcp | grep :0050
# Then look up the process by its inode
```

### /sys - Device and Kernel Information

**What it is:** Another virtual filesystem. Provides information about devices and allows you to configure hardware.

**What's inside:**

- Device information
- Hardware configuration
- Kernel module parameters

**Examples:**

```bash
# Information about your disks
ls /sys/block/

# Check if disk is rotational (HDD=1, SSD=0)
cat /sys/block/sda/queue/rotational

# Network interface information
ls /sys/class/net/

# Check if network interface is up
cat /sys/class/net/eth0/operstate

# MAC address
cat /sys/class/net/eth0/address
```

**Real-world scenario - Check if drive is SSD:**
```bash
# Check all drives
for drive in /sys/block/sd*; do
    echo -n "$(basename $drive): "
    cat $drive/queue/rotational
done

# Output:
# sda: 1  (HDD)
# sdb: 0  (SSD)
```

---

## Working with Files and Directories

### ls - List Files

**What it does:** Shows you what files and directories exist in a location.

**Why use it:** It's usually your first command when exploring a directory - "what's here?"

**Examples:**

```bash
# Basic list
ls

# Detailed list with permissions, owner, size, date
ls -l

# Show hidden files (names starting with .)
ls -a

# Human-readable sizes
ls -lh

# Sort by modification time, newest first
ls -lt

# Sort by size, largest first
ls -lS

# Show everything, sorted by time, human-readable
ls -lath

# List a specific directory without entering it
ls -l /var/log
```

**Understanding ls -l output:**
```bash
ls -l myfile.txt
-rw-r--r-- 1 john users 1024 Oct 28 10:30 myfile.txt
│││││││││  │ │    │     │    │           └─ filename
│││││││││  │ │    │     │    └─ date modified
│││││││││  │ │    │     └─ size (bytes)
│││││││││  │ │    └─ group owner
│││││││││  │ │─ user owner
│││││││││  └─ number of hard links
│└┴┴┴┴┴┴┴─ permissions
└─ file type (- = regular file, d = directory, l = link)
```

**File type indicators:**

- `-` Regular file
- `d` Directory
- `l` Symbolic link
- `b` Block device (like hard drives)
- `c` Character device (like terminals)

### cd - Change Directory

**What it does:** Moves you to a different directory.

**Why use it:** Navigation! You need to be in the right place to work on files.

**Examples:**

```bash
# Go to /var/log
cd /var/log

# Go to your home directory
cd ~
# or just
cd

# Go back to previous directory
cd -

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to a subdirectory of current location
cd ./subdir

# Relative to home
cd ~/Documents
```

**Real-world scenario:**
```bash
# Working in /var/www/html
cd /var/www/html

# Need to edit config in /etc
cd /etc/nginx

# Want to go back
cd -
# Now back in /var/www/html
```

### pwd - Print Working Directory

**What it does:** Shows you where you currently are in the filesystem.

**Why use it:** Ever get lost? `pwd` tells you exactly where you are.

**Examples:**

```bash
# Where am I?
pwd
# Output: /home/john/Documents

# After changing directories
cd /var/log
pwd
# Output: /var/log
```

### mkdir - Make Directory

**What it does:** Creates new directories.

**Why use it:** You need a place to organize your files!

**Examples:**

```bash
# Create a single directory
mkdir projects

# Create nested directories (including parents)
mkdir -p work/2024/reports

# Create multiple directories
mkdir dir1 dir2 dir3

# Create with specific permissions
mkdir -m 755 public_html

# Create project structure
mkdir -p myproject/{src,bin,docs,tests}
```

**Real-world scenario - Project setup:**
```bash
# Set up a new web project
mkdir -p ~/projects/mywebsite/{html,css,js,images}

# Result:
# mywebsite/
# ├── html/
# ├── css/
# ├── js/
# └── images/
```

### rm - Remove Files

**What it does:** Deletes files and directories.

**Why use it:** Clean up unwanted files, free up space.

**⚠️ WARNING:** There's no "Recycle Bin" in Linux command line. Deleted = GONE!

**Examples:**

```bash
# Delete a file
rm oldfile.txt

# Delete multiple files
rm file1.txt file2.txt file3.txt

# Delete a directory and its contents
rm -r oldfolder

# Force delete without asking
rm -f stubborn.txt

# Delete directory and everything in it (dangerous!)
rm -rf directory

# Interactive mode - asks before each deletion
rm -i *.txt

# Delete all .log files older than 30 days
find /var/log -name "*.log" -mtime +30 -exec rm {} \;
```

**Real-world scenario - Clean up old logs:**
```bash
# Find large log files
find /var/log -type f -size +100M

# Delete them (carefully!)
find /var/log -type f -size +100M -exec rm {} \;
```

**Safety tip:** Use `rm -i` (interactive) when deleting important files:
```bash
rm -i important*
# rm: remove regular file 'important-data.txt'? n (you type 'n' to keep it)
```

### cp - Copy Files

**What it does:** Makes a copy of files or directories.

**Why use it:** Backups, duplicating files, copying to different locations.

**Examples:**

```bash
# Copy a file
cp original.txt copy.txt

# Copy to a different directory
cp file.txt /backup/

# Copy directory and all contents
cp -r folder/ /backup/

# Copy and preserve permissions, ownership, timestamps
cp -a /etc/nginx /backup/nginx-backup

# Copy multiple files to a directory
cp file1.txt file2.txt file3.txt /destination/

# Interactive - ask before overwriting
cp -i file.txt existing-file.txt

# Update - only copy if source is newer
cp -u *.txt /backup/
```

**Real-world scenario - Backup before editing:**
```bash
# Before editing important config
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup

# Make changes
vi /etc/nginx/nginx.conf

# If something breaks, restore
cp /etc/nginx/nginx.conf.backup /etc/nginx/nginx.conf
```

### mv - Move or Rename Files

**What it does:** Moves files to new locations OR renames them (it's the same operation).

**Why use it:** Organize files, rename things.

**Examples:**

```bash
# Rename a file
mv oldname.txt newname.txt

# Move to different directory
mv file.txt /backup/

# Move multiple files
mv file1.txt file2.txt file3.txt /destination/

# Rename a directory
mv old-folder new-folder

# Move and rename at the same time
mv /tmp/file.txt /home/john/newfile.txt

# Interactive mode
mv -i file.txt existing.txt
```

**Real-world scenario:**
```bash
# Organizing downloaded files
mv ~/Downloads/*.pdf ~/Documents/PDFs/
mv ~/Downloads/*.jpg ~/Pictures/
```

### find - Search for Files

**What it does:** Searches for files based on various criteria (name, size, date, etc.).

**Why use it:** "I know I saved that file somewhere..." - `find` will locate it!

**Examples:**

```bash
# Find files by name
find /home -name "report.pdf"

# Case-insensitive search
find /home -iname "*.PDF"

# Find all directories
find /var -type d

# Find all files
find /var -type f

# Find files larger than 100MB
find /home -type f -size +100M

# Find files modified in last 7 days
find /var/log -type f -mtime -7

# Find files NOT modified in last 30 days
find /tmp -type f -mtime +30

# Find and execute command on results
find /tmp -type f -mtime +30 -exec rm {} \;

# Find files by owner
find /home -user john

# Find files with specific permissions
find /var/www -type f -perm 0777
```

**Real-world scenario - Find large files:**
```bash
# Disk is full, find what's using space
find / -type f -size +1G 2>/dev/null

# Output:
# /var/log/huge.log (5GB)
# /home/john/video.mp4 (2GB)
```

**Real-world scenario - Find recently changed configs:**
```bash
# What configs changed in last hour?
find /etc -type f -mmin -60

# Useful after installing software to see what changed
```

---

## File Permissions

### Understanding Linux Permissions

Every file has three levels of permissions:

- **User (u):** The owner of the file
- **Group (g):** The group that owns the file  
- **Others (o):** Everyone else

Each level can have three types of permissions:

- **Read (r):** Can view the file/list directory
- **Write (w):** Can modify the file/add files to directory
- **Execute (x):** Can run the file/enter the directory

**Example:**
```bash
ls -l script.sh
-rwxr-xr-- 1 john developers 2048 Oct 28 10:30 script.sh
 │││││││││
 │││││││└┴─ others: r-- (read only)
 │││└┴┴─── group: r-x (read and execute)
 └┴┴───── user: rwx (read, write, execute)
```

### chmod - Change Permissions

**What it does:** Changes who can read, write, or execute a file.

**Why use it:** Control access to your files, make scripts executable.

**Examples using symbolic mode:**

```bash
# Make file executable for everyone
chmod +x script.sh

# Make file executable only for owner
chmod u+x script.sh

# Remove write permission for group and others
chmod go-w file.txt

# Add read permission for everyone
chmod a+r document.pdf

# Set exact permissions: owner=rwx, group=rx, others=r
chmod u=rwx,g=rx,o=r script.sh
```

**Examples using numeric mode:**

Each permission has a number:
- r (read) = 4
- w (write) = 2
- x (execute) = 1

Add them up for each group:

```bash
# 755 = rwxr-xr-x (common for executables)
chmod 755 script.sh

# 644 = rw-r--r-- (common for regular files)
chmod 644 document.txt

# 700 = rwx------ (only owner can do anything)
chmod 700 private-script.sh

# 600 = rw------- (only owner can read/write)
chmod 600 private-data.txt

# 777 = rwxrwxrwx (everyone can do everything - usually bad!)
chmod 777 shared.sh
```

**Real-world scenario - Make script executable:**
```bash
# Download a script
wget https://example.com/install.sh

# Try to run it
./install.sh
# Error: Permission denied

# Make it executable
chmod +x install.sh

# Now it works
./install.sh
```

**Real-world scenario - Secure private key:**
```bash
# SSH won't work if private key is too open
chmod 600 ~/.ssh/id_rsa

# SSH directory itself
chmod 700 ~/.ssh
```

### chown - Change Owner

**What it does:** Changes who owns a file or directory.

**Why use it:** Fix ownership issues, give files to different users.

**Examples:**

```bash
# Change owner to john
chown john file.txt

# Change owner and group
chown john:developers file.txt

# Change only group (or use chgrp)
chown :developers file.txt

# Change ownership recursively
chown -R www-data:www-data /var/www/html

# Change using UID instead of username
chown 1000:1000 file.txt
```

**Real-world scenario - Web server files:**
```bash
# Apache needs to own web files
chown -R www-data:www-data /var/www/mysite

# But you want to edit them too
# Add yourself to www-data group
usermod -a -G www-data john

# Make files group-writable
chmod -R g+w /var/www/mysite
```

---

## Links

### What Are Links?

Links are like shortcuts or pointers to files. Linux has two types:

**Symbolic Links (Soft Links):**

- Like shortcuts in Windows
- Points to a file by its path
- If original is deleted, link breaks
- Can link to directories
- Can cross filesystem boundaries

**Hard Links:**

- Direct reference to the file data on disk
- Multiple names for the same data
- If original is deleted, data still exists
- Cannot link to directories (usually)
- Must be on same filesystem

### ln - Create Links

**What it does:** Creates symbolic or hard links to files.

**Why use it:** Create shortcuts, maintain backward compatibility, organize files.

**Examples:**

```bash
# Create symbolic link
ln -s /path/to/original /path/to/link

# Create hard link
ln /path/to/original /path/to/hardlink

# Symbolic link to directory
ln -s /usr/share/docs ~/docs

# Force overwrite existing link
ln -sf /new/target /existing/link

# Create link in current directory
ln -s /var/log/syslog syslog
```

**Real-world scenario - Python versions:**
```bash
# You have Python 3.9 but some programs expect 'python3'
ln -s /usr/bin/python3.9 /usr/bin/python3

# Now both work
python3.9 --version
python3 --version
```

**Real-world scenario - Configuration management:**
```bash
# Keep configs in version control
mkdir ~/configs
ln -s ~/configs/vimrc ~/.vimrc
ln -s ~/configs/bashrc ~/.bashrc

# Now you can edit ~/configs and commit changes
```

**Checking links:**
```bash
# List shows links with ->
ls -l /usr/bin/python3
# lrwxrwxrwx 1 root root 9 Mar 13  2023 /usr/bin/python3 -> python3.9
```

---

## Checking Disk Usage

### df - Disk Free

**What it does:** Shows how much disk space is used and available on each filesystem.

**Why use it:** "Is my disk full?" - Quick check of space on all mounted filesystems.

**Examples:**

```bash
# Human-readable sizes
df -h

# Show filesystem type too
df -hT

# Check specific filesystem
df -h /home

# Show inode usage (number of files)
df -hi

# All filesystems including pseudo ones
df -ha
```

**Output example:**
```bash
df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   30G   18G  63% /
/dev/sdb1       200G  150G   40G  79% /data
tmpfs           8.0G  1.0M  8.0G   1% /run
```

**Real-world scenario - Disk full warning:**
```bash
df -h /
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        20G   19G  100M  99% /

# Critical! Find what's using space:
du -sh /* | sort -h
```

### du - Disk Usage

**What it does:** Shows how much disk space files and directories are using.

**Why use it:** "What's taking up all my space?" - Find large files and directories.

**Examples:**

```bash
# Size of current directory
du -sh .

# Size of each item in current directory
du -sh *

# Top level only, human readable
du -h --max-depth=1 /var

# Sort by size, largest last
du -sh /home/* | sort -h

# Sort by size, largest first
du -sh /home/* | sort -rh

# Find directories larger than 1GB
du -h /var | grep '^[0-9.]*G'

# Exclude certain patterns
du -sh --exclude='*.log' /var
```

**Real-world scenario - Find space hogs:**
```bash
# Check /var which is using lots of space
du -sh /var/*
# Output:
# 50M    /var/cache
# 15G    /var/log
# 100M   /var/lib
# ...

# /var/log is huge! What inside?
du -sh /var/log/* | sort -rh | head -5
# Output:
# 10G    /var/log/old-logs
# 3G     /var/log/syslog
# 2G     /var/log/auth.log
```

---

## Quick Reference

### Navigation

```bash
pwd                    # Where am I?
cd /path              # Go to path
cd                    # Go home
cd -                  # Go back
ls -lah               # List files
```

### File Operations

```bash
mkdir dirname         # Create directory
rm file               # Delete file
rm -r dir             # Delete directory
cp source dest        # Copy
mv source dest        # Move/rename
find / -name file     # Search for file
```

### Permissions

```bash
chmod 755 file        # Change permissions
chown user:group file # Change owner
ls -l                 # View permissions
```

### Disk Usage

```bash
df -h                 # Show disk space
du -sh directory      # Directory size
du -sh * | sort -h    # Sort by size
```

### System Information

```bash
cat /proc/cpuinfo     # CPU info
cat /proc/meminfo     # Memory info
cat /etc/os-release   # OS version
```
