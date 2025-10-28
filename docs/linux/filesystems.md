# Create, Manage, and Troubleshoot Filesystems

## Overview
This guide covers creating, managing, and troubleshooting various Linux filesystems including ext4, XFS, Btrfs, and others.

---

## Common Filesystem Types

### Filesystem Comparison

| Filesystem | Max File Size | Max Volume Size | Features | Use Cases |
|------------|---------------|-----------------|----------|-----------|
| ext4 | 16 TB | 1 EB | Journaling, backwards compatible | General purpose, boot partitions |
| XFS | 8 EB | 8 EB | High performance, online defrag | Large files, databases, NAS |
| Btrfs | 16 EB | 16 EB | CoW, snapshots, subvolumes | Advanced features, snapshots |
| FAT32 | 4 GB | 2 TB | Cross-platform | USB drives, compatibility |
| exFAT | 16 EB | 128 PB | Large files, cross-platform | External drives, modern USB |
| NTFS | 16 EB | 16 EB | Windows native | Dual-boot, Windows compatibility |

---

## Creating Filesystems

### mkfs - Make Filesystem (Generic)
```bash
mkfs -t type device
mkfs.ext4 /dev/sdb1
```
**Common Options:**
- `-t` : Filesystem type
- `-L` : Set label
- `-v` : Verbose

**Use Cases:**
- Create filesystem quickly
- Generic filesystem creation

**Examples:**
```bash
# Create ext4 filesystem
mkfs -t ext4 /dev/sdb1

# Or use specific command
mkfs.ext4 /dev/sdb1

# With label
mkfs.ext4 -L "DATA" /dev/sdb1
```

### mkfs.ext4 - Create ext4 Filesystem
```bash
mkfs.ext4 [options] device
```
**Common Options:**
- `-L` : Set filesystem label
- `-m` : Reserved blocks percentage (default 5%)
- `-i` : Bytes per inode ratio
- `-J` : Journal options
- `-E` : Extended options
- `-b` : Block size (1024, 2048, 4096)
- `-F` : Force creation

**Use Cases:**
- Standard Linux filesystem
- Boot and system partitions
- General purpose storage

**Examples:**
```bash
# Basic ext4 creation
mkfs.ext4 /dev/sdb1

# With label
mkfs.ext4 -L "backup" /dev/sdb1

# Set 1% reserved blocks (more usable space)
mkfs.ext4 -m 1 /dev/sdb1

# Specify block size
mkfs.ext4 -b 4096 /dev/sdb1

# With journal on external device
mkfs.ext4 -J device=/dev/sdc1 /dev/sdb1

# More inodes (for many small files)
mkfs.ext4 -i 8192 /dev/sdb1

# Force creation (overwrites existing)
mkfs.ext4 -F /dev/sdb1
```

### mkfs.xfs - Create XFS Filesystem
```bash
mkfs.xfs [options] device
```
**Common Options:**
- `-L` : Set filesystem label
- `-f` : Force creation
- `-i` : Inode options
- `-d` : Data section options
- `-m` : Metadata options
- `-b` : Block size

**Use Cases:**
- High-performance storage
- Large files and databases
- Enterprise environments

**Examples:**
```bash
# Basic XFS creation
mkfs.xfs /dev/sdb1

# With label
mkfs.xfs -L "database" /dev/sdb1

# Force creation
mkfs.xfs -f /dev/sdb1

# Specify block size (default 4096)
mkfs.xfs -b size=4096 /dev/sdb1

# Set inode size
mkfs.xfs -i size=512 /dev/sdb1
```

### mkfs.btrfs - Create Btrfs Filesystem
```bash
mkfs.btrfs [options] device
```
**Common Options:**
- `-L` : Set label
- `-m` : Metadata RAID level
- `-d` : Data RAID level
- `-f` : Force creation

**Use Cases:**
- Advanced features needed (snapshots, CoW)
- Multi-device setups
- Systems requiring subvolumes

**Examples:**
```bash
# Basic Btrfs creation
mkfs.btrfs /dev/sdb1

# With label
mkfs.btrfs -L "storage" /dev/sdb1

# Multi-device with RAID1
mkfs.btrfs -m raid1 -d raid1 /dev/sdb1 /dev/sdc1

# Force creation
mkfs.btrfs -f /dev/sdb1
```

### mkfs.vfat / mkfs.fat - Create FAT Filesystem
```bash
mkfs.vfat [options] device
```
**Common Options:**
- `-F` : FAT size (12, 16, 32)
- `-n` : Volume name
- `-I` : Allow whole disk
- `-v` : Verbose

**Use Cases:**
- USB flash drives
- UEFI boot partitions
- Cross-platform compatibility

**Examples:**
```bash
# Create FAT32
mkfs.vfat -F 32 /dev/sdb1

# With volume name
mkfs.vfat -F 32 -n "USBDRIVE" /dev/sdb1

# For UEFI boot partition
mkfs.vfat -F 32 /dev/sda1
```

### mkfs.exfat - Create exFAT Filesystem
```bash
mkfs.exfat [options] device
```
**Common Options:**
- `-n` : Volume name

**Use Cases:**
- Large external drives
- Files > 4GB
- Cross-platform (Windows, Mac, Linux)

**Examples:**
```bash
# Create exFAT
mkfs.exfat /dev/sdb1

# With volume name
mkfs.exfat -n "EXTERNAL" /dev/sdb1
```

### mkswap - Create Swap Area
```bash
mkswap device
```
**Common Options:**
- `-L` : Set label
- `-U` : Set UUID
- `-c` : Check for bad blocks

**Use Cases:**
- Create swap space
- Configure virtual memory

**Examples:**
```bash
# Create swap
mkswap /dev/sdb2

# With label
mkswap -L "swap1" /dev/sdb2

# Check for bad blocks
mkswap -c /dev/sdb2
```

---

## Mounting Filesystems

### mount - Mount Filesystem
```bash
mount [options] device directory
```
**Common Options:**
- `-t` : Filesystem type
- `-o` : Mount options
- `-a` : Mount all in /etc/fstab
- `-r` : Read-only
- `-w` : Read-write

**Common Mount Options (-o):**
- `defaults` : rw, suid, dev, exec, auto, nouser, async
- `ro` : Read-only
- `rw` : Read-write
- `noexec` : Prevent execution
- `nosuid` : Ignore SUID bits
- `nodev` : No device files
- `user` : Allow users to mount
- `noatime` : Don't update access times
- `nodiratime` : Don't update directory access times
- `relatime` : Update access times relatively
- `sync` : Synchronous I/O
- `async` : Asynchronous I/O
- `remount` : Remount with new options

**Use Cases:**
- Attach filesystems
- Access storage devices
- Mount network shares

**Examples:**
```bash
# Basic mount
mount /dev/sdb1 /mnt/data

# Specify filesystem type
mount -t ext4 /dev/sdb1 /mnt/data

# Mount with options
mount -o ro,noexec /dev/sdb1 /mnt/data

# Mount all from fstab
mount -a

# Mount read-only
mount -r /dev/sdb1 /mnt/data

# Remount with new options
mount -o remount,rw /mnt/data

# Mount with multiple options
mount -o defaults,noatime,nodiratime /dev/sdb1 /mnt/data

# Mount ISO file
mount -o loop file.iso /mnt/iso

# Mount bind (mount directory elsewhere)
mount --bind /source/dir /target/dir

# Mount with specific user/group
mount -o uid=1000,gid=1000 /dev/sdb1 /mnt/data
```

### umount - Unmount Filesystem
```bash
umount device|directory
```
**Common Options:**
- `-f` : Force unmount
- `-l` : Lazy unmount (detach now, cleanup later)
- `-a` : Unmount all
- `-t` : Unmount specific type

**Use Cases:**
- Safely detach filesystems
- Prepare for disk removal
- System maintenance

**Examples:**
```bash
# Unmount by device
umount /dev/sdb1

# Unmount by directory
umount /mnt/data

# Force unmount
umount -f /mnt/data

# Lazy unmount (useful if busy)
umount -l /mnt/data

# Unmount all external drives
umount -a -t vfat
```

### findmnt - Find Mounted Filesystems
```bash
findmnt [options]
```
**Common Options:**
- `-t` : Filter by type
- `-S` : Search by source device
- `-T` : Search by target mount point
- `-o` : Output columns
- `-J` : JSON output

**Use Cases:**
- List mounted filesystems
- Find mount point for device
- Verify mount options

**Examples:**
```bash
# List all mounts in tree format
findmnt

# Find mount point
findmnt /dev/sdb1

# Find filesystem at path
findmnt /mnt/data

# List specific filesystem type
findmnt -t ext4

# Show specific columns
findmnt -o SOURCE,TARGET,FSTYPE,OPTIONS
```

---

## Managing Filesystems

### tune2fs - Adjust ext2/ext3/ext4 Parameters
```bash
tune2fs [options] device
```
**Common Options:**
- `-l` : List filesystem info
- `-L` : Set label
- `-m` : Set reserved blocks percentage
- `-c` : Set max mount count before check
- `-i` : Set check interval
- `-e` : Set error behavior
- `-j` : Add journal (ext2 to ext3)
- `-O` : Set/clear filesystem features

**Use Cases:**
- Change filesystem parameters
- Adjust reserved space
- Set filesystem label
- Tune performance

**Examples:**
```bash
# List filesystem information
tune2fs -l /dev/sdb1

# Set filesystem label
tune2fs -L "backup" /dev/sdb1

# Set 1% reserved blocks (default 5%)
tune2fs -m 1 /dev/sdb1

# Disable fsck on boot
tune2fs -c 0 -i 0 /dev/sdb1

# Set max mount count before check
tune2fs -c 30 /dev/sdb1

# Set check interval (180 days)
tune2fs -i 180d /dev/sdb1

# Set error behavior to remount read-only
tune2fs -e remount-ro /dev/sdb1

# Add journal to ext2 (convert to ext3)
tune2fs -j /dev/sdb1

# Enable 64-bit feature
tune2fs -O 64bit /dev/sdb1
```

### xfs_admin - XFS Filesystem Administration
```bash
xfs_admin [options] device
```
**Common Options:**
- `-l` : Print label
- `-L` : Set label
- `-u` : Print UUID
- `-U` : Set UUID

**Use Cases:**
- Change XFS label
- Modify UUID

**Examples:**
```bash
# Show label
xfs_admin -l /dev/sdb1

# Set label
xfs_admin -L "database" /dev/sdb1

# Show UUID
xfs_admin -u /dev/sdb1

# Generate new UUID
xfs_admin -U generate /dev/sdb1
```

### btrfs - Btrfs Filesystem Management
```bash
btrfs [command] [options]
```
**Common Commands:**
- `filesystem show` : Show btrfs filesystems
- `filesystem df` : Show space usage
- `filesystem resize` : Resize filesystem
- `subvolume create` : Create subvolume
- `subvolume list` : List subvolumes
- `subvolume snapshot` : Create snapshot
- `device add` : Add device
- `device remove` : Remove device
- `balance` : Balance data across devices

**Use Cases:**
- Manage subvolumes
- Create snapshots
- Resize online
- Multi-device management

**Examples:**
```bash
# Show btrfs filesystems
btrfs filesystem show

# Show space usage
btrfs filesystem df /mnt/btrfs

# Create subvolume
btrfs subvolume create /mnt/btrfs/subvol1

# List subvolumes
btrfs subvolume list /mnt/btrfs

# Create snapshot
btrfs subvolume snapshot /mnt/btrfs/subvol1 /mnt/btrfs/snap1

# Resize filesystem
btrfs filesystem resize +10G /mnt/btrfs
btrfs filesystem resize max /mnt/btrfs

# Add device
btrfs device add /dev/sdc /mnt/btrfs

# Remove device
btrfs device remove /dev/sdc /mnt/btrfs

# Balance filesystem
btrfs balance start /mnt/btrfs
```

---

## Checking and Repairing Filesystems

### fsck - Filesystem Check (Generic)
```bash
fsck [options] device
```
**Common Options:**
- `-A` : Check all in /etc/fstab
- `-a` : Auto repair
- `-y` : Automatic yes to all
- `-n` : Check only (no repair)
- `-f` : Force check
- `-t` : Filesystem type
- `-C` : Display progress bar

**Use Cases:**
- Check filesystem integrity
- Repair filesystem errors
- Routine maintenance

**Examples:**
```bash
# Check filesystem (unmounted!)
fsck /dev/sdb1

# Auto repair
fsck -a /dev/sdb1

# Force check even if clean
fsck -f /dev/sdb1

# Check all in fstab
fsck -A

# Check with yes to all
fsck -y /dev/sdb1
```

**IMPORTANT:** Always unmount filesystem before running fsck!

### e2fsck - ext2/ext3/ext4 Filesystem Check
```bash
e2fsck [options] device
```
**Common Options:**
- `-p` : Auto repair safe problems
- `-y` : Automatic yes
- `-f` : Force check
- `-n` : Check only (no changes)
- `-v` : Verbose
- `-c` : Check for bad blocks

**Use Cases:**
- Repair ext filesystems
- Check for corruption
- Find bad blocks

**Examples:**
```bash
# Auto repair
e2fsck -p /dev/sdb1

# Force check with yes to all
e2fsck -fy /dev/sdb1

# Check for bad blocks
e2fsck -c /dev/sdb1

# Verbose check
e2fsck -fv /dev/sdb1

# Read-only check
e2fsck -n /dev/sdb1
```

### xfs_repair - Repair XFS Filesystem
```bash
xfs_repair [options] device
```
**Common Options:**
- `-n` : No modify (dry run)
- `-v` : Verbose
- `-d` : Dangerous mode
- `-L` : Force log zeroing

**Use Cases:**
- Repair XFS filesystem
- Recover from corruption

**Examples:**
```bash
# Dry run (check only)
xfs_repair -n /dev/sdb1

# Repair filesystem
xfs_repair /dev/sdb1

# Force log zeroing (last resort)
xfs_repair -L /dev/sdb1
```

**Note:** XFS has no auto-repair option. Must unmount before repair.

### btrfs check - Check Btrfs Filesystem
```bash
btrfs check [options] device
```
**Common Options:**
- `--repair` : Repair filesystem
- `--readonly` : Check only
- `--force` : Force check

**Use Cases:**
- Check btrfs integrity
- Repair btrfs filesystem

**Examples:**
```bash
# Check only (unmounted)
btrfs check /dev/sdb1

# Repair (dangerous, use scrub instead if possible)
btrfs check --repair /dev/sdb1

# Scrub online (preferred method)
btrfs scrub start /mnt/btrfs
btrfs scrub status /mnt/btrfs
```

### badblocks - Search for Bad Blocks
```bash
badblocks [options] device
```
**Common Options:**
- `-v` : Verbose
- `-w` : Write test (destructive!)
- `-n` : Non-destructive read-write test
- `-s` : Show progress

**Use Cases:**
- Find bad blocks on disk
- Test disk reliability
- Before using new disk

**Examples:**
```bash
# Non-destructive test
badblocks -v /dev/sdb

# Write test (DESTROYS DATA!)
badblocks -wv /dev/sdb

# Non-destructive read-write test
badblocks -nv /dev/sdb

# Save bad blocks list
badblocks -v /dev/sdb > badblocks.txt

# Use with e2fsck
badblocks -v /dev/sdb > /tmp/badblocks
e2fsck -l /tmp/badblocks /dev/sdb
```

---

## Resizing Filesystems

### resize2fs - Resize ext2/ext3/ext4
```bash
resize2fs device [size]
```
**Common Options:**
- `-p` : Print progress
- `-f` : Force resize
- `-M` : Minimize (shrink to minimum)

**Use Cases:**
- Resize after LV extend/reduce
- Grow filesystem to fill partition

**Examples:**
```bash
# Resize to fill partition (after extending)
resize2fs /dev/sdb1

# Resize to specific size
resize2fs /dev/sdb1 50G

# Shrink to minimum size
resize2fs -M /dev/sdb1

# With progress
resize2fs -p /dev/sdb1
```

**Workflow for Growing:**
```bash
# 1. Extend partition/LV
lvextend -L +10G /dev/vg0/lv_data

# 2. Resize filesystem
resize2fs /dev/vg0/lv_data
```

**Workflow for Shrinking:**
```bash
# 1. Check and repair
e2fsck -f /dev/vg0/lv_data

# 2. Shrink filesystem FIRST
resize2fs /dev/vg0/lv_data 20G

# 3. Shrink LV to match
lvreduce -L 20G /dev/vg0/lv_data
```

### xfs_growfs - Grow XFS Filesystem
```bash
xfs_growfs mountpoint
```
**Common Options:**
- `-d` : Grow data section
- `-D` : Grow to specific size

**Use Cases:**
- Expand XFS filesystem (online)

**Examples:**
```bash
# Grow to fill device
xfs_growfs /mnt/data

# Grow by specific size
xfs_growfs -D 100g /mnt/data
```

**Note:** XFS **cannot** be shrunk! Plan size carefully.

### btrfs filesystem resize - Resize Btrfs
```bash
btrfs filesystem resize [size] mountpoint
```
**Use Cases:**
- Resize btrfs online

**Examples:**
```bash
# Grow by 10GB
btrfs filesystem resize +10G /mnt/btrfs

# Shrink by 5GB
btrfs filesystem resize -5G /mnt/btrfs

# Grow to maximum
btrfs filesystem resize max /mnt/btrfs
```

---

## Filesystem Information

### blkid - Block Device Attributes
```bash
blkid [device]
```
**Common Options:**
- `-o` : Output format
- `-s` : Show specific tag

**Use Cases:**
- Get UUID and filesystem type
- Identify block devices

**Examples:**
```bash
# Show all block devices
blkid

# Show specific device
blkid /dev/sdb1

# Show only UUID
blkid -s UUID /dev/sdb1

# Output as list
blkid -o list
```

### lsblk - List Block Devices
```bash
lsblk [options]
```
**Common Options:**
- `-f` : Show filesystem info
- `-o` : Output columns
- `-p` : Full device paths
- `-a` : Show all devices

**Use Cases:**
- View block device tree
- Check filesystem types
- See mount points

**Examples:**
```bash
# Basic tree view
lsblk

# With filesystem info
lsblk -f

# Custom columns
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT

# Full paths
lsblk -p
```

### dumpe2fs - Dump ext Filesystem Information
```bash
dumpe2fs [options] device
```
**Common Options:**
- `-h` : Show superblock info only
- `-b` : Show bad blocks

**Use Cases:**
- Detailed ext filesystem info
- View superblock data

**Examples:**
```bash
# Show all info
dumpe2fs /dev/sdb1

# Superblock only
dumpe2fs -h /dev/sdb1

# Show bad blocks
dumpe2fs -b /dev/sdb1
```

---

## Troubleshooting

### Common Issues and Solutions

#### 1. Filesystem Won't Mount
```bash
# Check filesystem type
blkid /dev/sdb1
lsblk -f

# Try mounting with explicit type
mount -t ext4 /dev/sdb1 /mnt/data

# Check for errors
dmesg | tail
journalctl -xe
```

#### 2. Device or Resource Busy
```bash
# Find what's using it
lsof /mnt/data
fuser -m /mnt/data

# Kill processes (careful!)
fuser -km /mnt/data

# Lazy unmount
umount -l /mnt/data
```

#### 3. Filesystem Corruption
```bash
# Unmount first!
umount /dev/sdb1

# Check and repair
fsck -fy /dev/sdb1

# For ext filesystems
e2fsck -fy /dev/sdb1

# For XFS
xfs_repair /dev/sdb1
```

#### 4. Lost UUID After Clone
```bash
# Generate new UUID (ext)
tune2fs -U random /dev/sdb1

# For XFS
xfs_admin -U generate /dev/sdb1

# Verify
blkid /dev/sdb1
```

#### 5. Read-only Filesystem
```bash
# Remount read-write
mount -o remount,rw /mount/point

# Check for errors
dmesg | grep -i "read-only"

# May need fsck
umount /dev/sdb1
fsck -y /dev/sdb1
mount /dev/sdb1 /mount/point
```

#### 6. No Space Left on Device (But df Shows Space)
```bash
# Check inodes
df -i

# If inodes full, delete many small files
find /path -type f -size 0 -delete

# Or increase inode count (requires recreation)
```

#### 7. Bad Superblock
```bash
# Show backup superblocks
dumpe2fs /dev/sdb1 | grep -i superblock

# Use backup superblock
e2fsck -b 32768 /dev/sdb1
```

### Diagnostic Commands

#### Check Mount Status
```bash
mount | grep sdb
findmnt /dev/sdb1
cat /proc/mounts | grep sdb
```

#### Check Filesystem Usage
```bash
df -h
df -i
du -sh /path/*
```

#### Monitor Filesystem Events
```bash
dmesg | grep -i "ext4\|xfs\|btrfs"
journalctl -f | grep mount
```

#### Check for Bad Blocks
```bash
badblocks -v /dev/sdb1
smartctl -t long /dev/sdb    # SMART test
smartctl -a /dev/sdb         # SMART data
```

---

## Best Practices

1. **Always unmount** before fsck
2. **Backup data** before resizing
3. **Shrink filesystem BEFORE** reducing partition
4. **Extend partition BEFORE** growing filesystem
5. **Use appropriate filesystem** for use case
6. **Set proper labels** for easy identification
7. **Check filesystems regularly** (schedule checks)
8. **Monitor disk health** with SMART
9. **Test recovery procedures** before disaster
10. **Document custom mount options** in /etc/fstab

---

## Quick Reference

### Filesystem Creation
```bash
mkfs.ext4 /dev/sdb1                    # ext4
mkfs.xfs /dev/sdb1                     # XFS
mkfs.btrfs /dev/sdb1                   # Btrfs
mkfs.vfat -F 32 /dev/sdb1              # FAT32
```

### Mount/Unmount
```bash
mount /dev/sdb1 /mnt/data              # Mount
umount /mnt/data                       # Unmount
mount -a                               # Mount all (fstab)
```

### Check/Repair
```bash
fsck -y /dev/sdb1                      # Generic check
e2fsck -fy /dev/sdb1                   # ext check
xfs_repair /dev/sdb1                   # XFS repair
```

### Resize
```bash
resize2fs /dev/sdb1                    # ext grow/shrink
xfs_growfs /mnt/data                   # XFS grow only
btrfs filesystem resize +10G /mnt      # Btrfs grow/shrink
```

### Information
```bash
blkid                                  # UUIDs and types
lsblk -f                               # Block device tree
df -h                                  # Space usage
df -i                                  # Inode usage
tune2fs -l /dev/sdb1                   # ext info
```
