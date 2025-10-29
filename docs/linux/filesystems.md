# Create, Manage, and Troubleshoot Filesystems

## What is a Filesystem?

A filesystem is the method and structure used to organize and store files on a storage device. Think of it as the filing system for your hard drive - it determines how data is stored, how files are named, what metadata is kept, and how space is managed.

Without a filesystem, a hard drive is just a blank slate. The filesystem gives it structure and makes it usable.

### Why Different Filesystems?

Different filesystems are optimized for different uses:

- **ext4:** General purpose, very stable, great for Linux systems
- **XFS:** Excellent for large files, high performance, used in enterprise
- **Btrfs:** Modern features like snapshots and compression
- **VFAT/FAT32:** Cross-platform compatibility (Windows, Mac, Linux)
- **NTFS:** Windows native filesystem
- **exFAT:** Modern cross-platform for large files

---

## Common Linux Filesystems

### ext4 - Fourth Extended Filesystem

**What it is:** The most common Linux filesystem. Reliable, mature, and the default on most Linux distributions.

**Best for:**

- Linux system partitions
- General purpose storage
- Root and /home partitions
- When you need stability and maturity

**Key features:**

- Maximum file size: 16 TB
- Maximum partition size: 1 EB (Exabyte)
- Journaling (protects against corruption)
- Backwards compatible with ext3 and ext2

**When to use:** Default choice for most Linux installations. If unsure, use ext4.

### XFS - High Performance Filesystem

**What it is:** Originally from SGI, designed for high-performance computing.

**Best for:**

- Large files (videos, databases, disk images)
- High throughput workloads
- Enterprise storage systems
- NAS devices

**Key features:**

- Excellent performance with large files
- Online defragmentation
- Can grow online (but cannot shrink!)
- Maximum file size: 8 EB

**When to use:** Database servers, video editing, large file storage.

**Important limitation:** You CANNOT shrink XFS filesystems!

### Btrfs - B-Tree Filesystem

**What it is:** Modern filesystem with advanced features like built-in snapshots and compression.

**Best for:**

- When you need snapshots
- When you want compression
- Desktop systems
- Development environments

**Key features:**

- Built-in snapshots (no LVM needed)
- Online compression
- Self-healing (detects and repairs corruption)
- Subvolumes (like lightweight partitions)

**When to use:** When you want modern features and don't mind some complexity.

### FAT32 and exFAT

**What they are:** Simple filesystems that work on Windows, Mac, and Linux.

**Best for:**

- USB flash drives
- External hard drives
- Sharing files between different operating systems

**Limitations:**

- **FAT32:** Cannot handle files larger than 4GB
- **exFAT:** No file size limit, but not as widely supported as FAT32

---

## Creating Filesystems

### mkfs.ext4 - Create ext4 Filesystem

**What it does:** Formats a partition or disk with the ext4 filesystem.

**Why use it:** To make a disk usable for storing files on Linux.

**⚠️ WARNING:** This ERASES all data on the partition!

**Examples:**

```bash
# Basic ext4 creation
mkfs.ext4 /dev/sdb1

# With a label (helps identify the filesystem)
mkfs.ext4 -L "DATA" /dev/sdb1

# With less reserved space (default is 5%, which wastes space)
mkfs.ext4 -m 1 /dev/sdb1

# Force creation (even if filesystem already exists)
mkfs.ext4 -F /dev/sdb1
```

**Real-world scenario - New data drive:**

```bash
# You added a new 1TB drive to your server
# Step 1: Create partition
fdisk /dev/sdb
# Press: n, p, 1, Enter, Enter, w

# Step 2: Create ext4 filesystem with label
mkfs.ext4 -L "ServerData" /dev/sdb1

# Output:
# Creating filesystem with 262144000 4k blocks and 65536000 inodes
# Filesystem UUID: 1234abcd-5678-efgh...
# Allocating group tables: done
# Writing inode tables: done
# Creating journal (32768 blocks): done
# Writing superblocks and filesystem accounting information: done

# Step 3: Create mount point and mount
mkdir /data
mount /dev/sdb1 /data

# Step 4: Make it permanent
echo "LABEL=ServerData /data ext4 defaults 0 2" >> /etc/fstab
```

**Understanding the output:**

- **UUID:** Unique identifier for this filesystem
- **Inode tables:** Where file metadata is stored
- **Journal:** Protects against corruption during crashes

### mkfs.xfs - Create XFS Filesystem

**What it does:** Formats a partition with the XFS filesystem.

**Why use it:** When you need high performance for large files or databases.

**Examples:**

```bash
# Basic XFS creation
mkfs.xfs /dev/sdb1

# With a label
mkfs.xfs -L "Database" /dev/sdb1

# Force creation (overwrite existing)
mkfs.xfs -f /dev/sdb1
```

**Real-world scenario - Database server:**

```bash
# Creating XFS for PostgreSQL database
mkfs.xfs -L "PostgreSQL" /dev/sdb1

# Mount it
mkdir /var/lib/postgresql
mount /dev/sdb1 /var/lib/postgresql

# Permanent mount
echo "LABEL=PostgreSQL /var/lib/postgresql xfs defaults 0 2" >> /etc/fstab

# Change ownership for PostgreSQL
chown -R postgres:postgres /var/lib/postgresql
```

### mkfs.btrfs - Create Btrfs Filesystem

**What it does:** Creates a Btrfs filesystem with advanced features.

**Why use it:** When you want snapshots, compression, or other modern features.

**Examples:**

```bash
# Basic Btrfs creation
mkfs.btrfs /dev/sdb1

# With a label
mkfs.btrfs -L "BtrfsData" /dev/sdb1

# Create with multiple devices (RAID-like)
mkfs.btrfs -d raid1 -m raid1 /dev/sdb1 /dev/sdc1

# Force creation
mkfs.btrfs -f /dev/sdb1
```

**Real-world scenario - Development workstation:**

```bash
# Create Btrfs for /home with snapshot capability
mkfs.btrfs -L "Home" /dev/sdb1

# Mount with compression
mount -o compress=zstd /dev/sdb1 /home

# Make permanent with compression
echo "LABEL=Home /home btrfs compress=zstd 0 0" >> /etc/fstab
```

### mkfs.vfat - Create FAT32 Filesystem

**What it does:** Creates FAT32 filesystem for cross-platform compatibility.

**Why use it:** USB drives, EFI boot partitions, sharing between operating systems.

**Examples:**

```bash
# Create FAT32
mkfs.vfat -F 32 /dev/sdb1

# With volume name
mkfs.vfat -F 32 -n "USB_DRIVE" /dev/sdb1
```

**Real-world scenario - USB flash drive:**

```bash
# Format USB drive for Windows/Mac/Linux compatibility
# Step 1: Identify the drive
lsblk
# sdb is your USB drive

# Step 2: Unmount if mounted
umount /dev/sdb1

# Step 3: Create new partition (if needed)
fdisk /dev/sdb
# d (delete), n (new), p (primary), 1, Enter, Enter, t (type), c (W95 FAT32), w (write)

# Step 4: Format as FAT32
mkfs.vfat -F 32 -n "MYUSB" /dev/sdb1

# Now it works on any computer
```

---

## Mounting Filesystems

### Understanding Mounting

In Linux, you don't access drives by letters like Windows (C:, D:). Instead, you "mount" them to directories. The mount point is where the filesystem appears in your directory tree.

**Example:** Mount `/dev/sdb1` to `/data`, and the files on that disk appear at `/data`.

### mount - Attach Filesystems

**What it does:** Makes a filesystem accessible at a specific directory.

**Why use it:** You can't access files on a disk until it's mounted!

**Examples:**

```bash
# Basic mount
mount /dev/sdb1 /mnt

# Mount with filesystem type specified
mount -t ext4 /dev/sdb1 /mnt/data

# Mount read-only
mount -o ro /dev/sdb1 /mnt/data

# Mount with multiple options
mount -o rw,noatime /dev/sdb1 /mnt/data

# Mount all entries in /etc/fstab
mount -a

# Remount with different options (no unmount needed)
mount -o remount,rw /mnt/data

# Mount an ISO file
mount -o loop image.iso /mnt/iso

# Bind mount (mount directory to another location)
mount --bind /source/dir /dest/dir
```

**Common mount options:**

- `ro` - Read-only (cannot modify files)
- `rw` - Read-write (can modify files) 
- `noatime` - Don't update access times (faster)
- `noexec` - Cannot execute binaries (security)
- `nosuid` - Ignore setuid bits (security)
- `nodev` - No device files (security)

**Real-world scenario - Mounting a new drive:**

```bash
# You've created filesystem on /dev/sdb1
# Create mount point
mkdir /data

# Test mount first
mount /dev/sdb1 /data

# Check if it worked
df -h /data
ls /data

# If good, make permanent
echo "/dev/sdb1 /data ext4 defaults 0 2" >> /etc/fstab

# Unmount
umount /data

# Test fstab entry
mount -a

# Verify
df -h /data
```

**Real-world scenario - Mount with specific options:**

```bash
# Mount external drive with better performance
mount -o noatime,nodiratime /dev/sdb1 /media/external

# Mount with specific user ownership (for FAT32/exFAT)
mount -o uid=1000,gid=1000 /dev/sdb1 /media/usb
```

### umount - Detach Filesystems

**What it does:** Unmounts a filesystem, making it safe to remove.

**Why use it:** Before unplugging USB drives or doing maintenance, always unmount!

**Examples:**

```bash
# Unmount by mount point
umount /mnt/data

# Unmount by device
umount /dev/sdb1

# Force unmount (if regular umount fails)
umount -f /mnt/data

# Lazy unmount (detach now, cleanup when no longer busy)
umount -l /mnt/data

# Unmount all
umount -a
```

**Real-world scenario - "Device is busy" error:**

```bash
# Try to unmount
umount /mnt/data
# Error: target is busy

# Find what's using it
lsof /mnt/data
# Shows: some-process is using a file

# or
fuser -m /mnt/data
# Shows: 1234 (PID of process)

# Option 1: Close the application
kill 1234

# Option 2: Force unmount
umount -f /mnt/data

# Option 3: Lazy unmount (not recommended unless necessary)
umount -l /mnt/data
```

### /etc/fstab - Permanent Mounts

**What it is:** Configuration file that tells Linux what to mount automatically at boot.

**Why use it:** So you don't have to manually mount filesystems every time you reboot.

**Format:**

```
device  mount_point  filesystem_type  options  dump  pass
```

**Examples:**

```bash
# Basic entry for data partition
/dev/sdb1  /data  ext4  defaults  0  2

# Using UUID (more reliable than /dev/sdX)
UUID=1234-5678  /data  ext4  defaults  0  2

# Using label
LABEL=DATA  /data  ext4  defaults  0  2

# Network filesystem (wait for network)
192.168.1.100:/share  /mnt/nfs  nfs  defaults,_netdev  0  0

# Swap partition
/dev/sdb2  none  swap  sw  0  0

# With performance options
UUID=abcd-1234  /data  ext4  noatime,nodiratime  0  2

# USB drive that might not always be present
UUID=1234-5678  /media/usb  vfat  noauto,user  0  0
```

**Understanding the fields:**

1. **Device:** `/dev/sdb1`, `UUID=...`, or `LABEL=...`
2. **Mount point:** Where it appears (`/data`, `/mnt/backup`)
3. **Filesystem:** `ext4`, `xfs`, `ntfs`, `vfat`, etc.
4. **Options:** `defaults`, `noatime`, `ro`, etc.
5. **Dump:** Backup with dump (0=no, 1=yes) - rarely used
6. **Pass:** fsck check order (0=no check, 1=first, 2=later)

**Real-world scenario - Adding entry:**

```bash
# Get UUID of partition
blkid /dev/sdb1
# Output: /dev/sdb1: UUID="1234abcd-..." TYPE="ext4" LABEL="DATA"

# Edit fstab
vi /etc/fstab

# Add entry (using UUID is more reliable)
UUID=1234abcd-... /data ext4 defaults 0 2

# Save and test (doesn't actually mount, just checks syntax)
mount -a

# If no errors, reboot to verify it mounts automatically
reboot
```

---

## Managing Filesystems

### tune2fs - Tune ext4 Filesystems

**What it does:** Adjusts parameters of ext2/ext3/ext4 filesystems.

**Why use it:** Change labels, reserved space, mount counts, and other settings without reformatting.

**Examples:**

```bash
# View filesystem information
tune2fs -l /dev/sdb1

# Change filesystem label
tune2fs -L "NewLabel" /dev/sdb1

# Set reserved space to 1% (default is 5%)
tune2fs -m 1 /dev/sdb1

# Disable fsck on mount count
tune2fs -c 0 /dev/sdb1

# Disable fsck on time interval
tune2fs -i 0 /dev/sdb1

# Set error behavior to remount read-only
tune2fs -e remount-ro /dev/sdb1
```

**Real-world scenario - Free up reserved space:**

```bash
# ext4 reserves 5% for root by default
# On large data drives, this wastes space

# Check current settings
tune2fs -l /dev/sdb1 | grep -i "block count\|reserved"

# Reserved block count:     5120000 (5% of 100GB = 5GB wasted!)
# Change to 1%
tune2fs -m 1 /dev/sdb1

# Now you have 4GB more usable space
```

### xfs_admin - Manage XFS Filesystems

**What it does:** Changes parameters of XFS filesystems.

**Why use it:** Change labels or UUIDs on XFS.

**Examples:**

```bash
# Show current label
xfs_admin -l /dev/sdb1

# Set new label
xfs_admin -L "NewLabel" /dev/sdb1

# Show UUID
xfs_admin -u /dev/sdb1

# Generate new UUID
xfs_admin -U generate /dev/sdb1
```

### btrfs - Manage Btrfs Filesystems

**What it does:** Comprehensive tool for managing Btrfs features.

**Why use it:** Work with subvolumes, snapshots, and other Btrfs features.

**Examples:**

```bash
# Show Btrfs filesystems
btrfs filesystem show

# Show space usage
btrfs filesystem df /mnt/btrfs

# Create subvolume
btrfs subvolume create /mnt/btrfs/mysubvol

# List subvolumes
btrfs subvolume list /mnt/btrfs

# Create snapshot
btrfs subvolume snapshot /mnt/btrfs/mysubvol /mnt/btrfs/snapshot_2024

# Resize filesystem (can grow or shrink)
btrfs filesystem resize +10G /mnt/btrfs
btrfs filesystem resize max /mnt/btrfs
```

**Real-world scenario - Snapshots before update:**

```bash
# Take snapshot before system update
btrfs subvolume snapshot / /.snapshots/before-update

# Perform update
dnf update -y

# If something breaks, boot from snapshot
# (requires bootloader configuration)

# If everything works, delete snapshot
btrfs subvolume delete /.snapshots/before-update
```

---

## Checking and Repairing Filesystems

### fsck - Filesystem Check

**What it does:** Checks and repairs filesystem errors.

**Why use it:** Fix corruption, prepare for resize operations, routine maintenance.

**⚠️ CRITICAL:** Always unmount before running fsck!

**Examples:**

```bash
# Check filesystem (unmount first!)
fsck /dev/sdb1

# Automatic repair
fsck -y /dev/sdb1

# Check even if marked clean
fsck -f /dev/sdb1

# Check all filesystems in /etc/fstab
fsck -A
```

**Real-world scenario - Fixing corruption:**

```bash
# System won't boot, dropped to emergency shell
# Root filesystem has errors

# Try to mount, fails with errors
mount /dev/sda1 /mnt
# Error: filesystem has errors

# Check and repair
fsck -y /dev/sda1
# Pass 1: Checking inodes...
# Pass 2: Checking directory structure...
# Pass 3: Checking directory connectivity...
# Pass 4: Checking reference counts...
# Pass 5: Checking group summary information...
# Fixed 15 errors

# Now mount works
mount /dev/sda1 /mnt
```

### e2fsck - ext4 Filesystem Check

**What it does:** Specialized tool for ext2/ext3/ext4 filesystems.

**Why use it:** More control than generic fsck for ext filesystems.

**Examples:**

```bash
# Automatic repair
e2fsck -y /dev/sdb1

# Force check even if clean
e2fsck -f /dev/sdb1

# Check for bad blocks
e2fsck -c /dev/sdb1

# Read-only check (no modifications)
e2fsck -n /dev/sdb1
```

### xfs_repair - XFS Filesystem Repair

**What it does:** Repairs XFS filesystems.

**Why use it:** XFS doesn't use fsck; use this instead.

**Examples:**

```bash
# Dry run (check only)
xfs_repair -n /dev/sdb1

# Repair filesystem (unmount first!)
xfs_repair /dev/sdb1

# Force log zeroing (last resort, may lose data)
xfs_repair -L /dev/sdb1
```

---

## Resizing Filesystems

### resize2fs - Resize ext4

**What it does:** Grows or shrinks ext2/ext3/ext4 filesystems.

**Why use it:** Adjust filesystem size to match partition size.

**Examples:**

```bash
# Grow to fill partition (after extending partition)
resize2fs /dev/sdb1

# Shrink to specific size (unmount first!)
resize2fs /dev/sdb1 50G

# Shrink to minimum possible size
resize2fs -M /dev/sdb1
```

**Real-world scenario - Growing filesystem:**

```bash
# You extended LVM volume from 50GB to 100GB
# Filesystem is still 50GB

# Step 1: Check current size
df -h /data
# 50GB

# Step 2: Resize filesystem
resize2fs /dev/vg0/lv_data

# Step 3: Verify
df -h /data
# 100GB - done!
```

**Real-world scenario - Shrinking filesystem:**

```bash
# You want to shrink partition from 100GB to 50GB
# ALWAYS shrink filesystem BEFORE shrinking partition!

# Step 1: Unmount
umount /data

# Step 2: Check filesystem
e2fsck -f /dev/sdb1

# Step 3: Shrink filesystem to 50GB
resize2fs /dev/sdb1 50G

# Step 4: Shrink partition to 50GB (using fdisk, parted, or LVM)
lvreduce -L 50G /dev/vg0/lv_data

# Step 5: Mount
mount /dev/vg0/lv_data /data
```

### xfs_growfs - Grow XFS

**What it does:** Grows XFS filesystems (online - while mounted).

**Why use it:** Expand XFS to use additional space.

**⚠️ Note:** XFS can ONLY grow, never shrink!

**Examples:**

```bash
# Grow to fill device
xfs_growfs /mnt/data

# Grow by specific amount
xfs_growfs -D 100g /mnt/data
```

**Real-world scenario:**

```bash
# Extended partition, need to grow XFS
# Step 1: Extend partition (already done)

# Step 2: Grow filesystem (while mounted!)
xfs_growfs /data

# Step 3: Verify
df -h /data
```

---

## Checking Filesystem Information

### blkid - Block Device Information

**What it does:** Shows UUID, label, and filesystem type of block devices.

**Why use it:** Get information needed for /etc/fstab entries.

**Examples:**

```bash
# Show all block devices
blkid

# Show specific device
blkid /dev/sdb1

# Show only UUID
blkid -s UUID -o value /dev/sdb1

# Show only label
blkid -s LABEL -o value /dev/sdb1
```

**Output example:**

```bash
blkid /dev/sdb1
/dev/sdb1: UUID="1234abcd-..." TYPE="ext4" LABEL="DATA" PARTLABEL="primary" PARTUUID="..."
```

### lsblk - List Block Devices

**What it does:** Shows tree view of all block devices with mount points.

**Why use it:** Quick overview of all disks and partitions.

**Examples:**

```bash
# Basic tree view
lsblk

# With filesystem information
lsblk -f

# With full device paths
lsblk -p

# Custom columns
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
```

**Output example:**

```bash
lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  100G  0 disk
├─sda1   8:1    0   50G  0 part /
└─sda2   8:2    0   50G  0 part /home
sdb      8:16   0  500G  0 disk
└─sdb1   8:17   0  500G  0 part /data
```

---

## Troubleshooting

### Problem: Filesystem Won't Mount

**Symptoms:** Mount command fails with errors.

**Solutions:**

```bash
# Check if device exists
ls -l /dev/sdb1

# Check filesystem type
blkid /dev/sdb1

# Try specifying filesystem type
mount -t ext4 /dev/sdb1 /mnt

# Check for errors
fsck -y /dev/sdb1

# Check system logs
dmesg | tail -20
journalctl -xe
```

### Problem: Device is Busy

**Symptoms:** Cannot unmount because device is in use.

**Solutions:**

```bash
# Find what's using it
lsof /mnt/data
fuser -vm /mnt/data

# Change to different directory
cd ~

# Kill processes (careful!)
fuser -k /mnt/data

# Lazy unmount (last resort)
umount -l /mnt/data
```

### Problem: Read-only Filesystem

**Symptoms:** Cannot write to filesystem.

**Solutions:**

```bash
# Remount as read-write
mount -o remount,rw /mount/point

# If that fails, check for errors
umount /mount/point
fsck -y /dev/sdb1
mount /dev/sdb1 /mount/point
```

### Problem: No Space Left on Device (But df Shows Space)

**Symptoms:** Cannot create files even though df shows free space.

**Cause:** Out of inodes (file metadata structures).

**Solution:**

```bash
# Check inodes
df -i

# If inodes are at 100%, delete many small files
# Or recreate filesystem with more inodes
```

---

## Quick Reference

### Creating Filesystems

```bash
mkfs.ext4 /dev/sdb1                # ext4
mkfs.xfs /dev/sdb1                 # XFS
mkfs.btrfs /dev/sdb1               # Btrfs
mkfs.vfat -F 32 /dev/sdb1          # FAT32
```

### Mounting

```bash
mount /dev/sdb1 /mnt               # Mount
umount /mnt                        # Unmount
mount -a                           # Mount all (fstab)
```

### Checking

```bash
fsck -y /dev/sdb1                  # Check and repair
e2fsck -f /dev/sdb1                # ext4 check
xfs_repair /dev/sdb1               # XFS repair
```

### Resizing

```bash
resize2fs /dev/sdb1                # Grow ext4
xfs_growfs /mnt/data               # Grow XFS
```

### Information

```bash
blkid /dev/sdb1                    # UUID, label, type
lsblk -f                           # Tree with fs info
df -h                              # Space usage
tune2fs -l /dev/sdb1               # ext4 details
```
