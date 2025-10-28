# Configure and Manage LVM Storage

## What is LVM?

LVM (Logical Volume Manager) is a flexible disk management system that sits between your physical hard drives and your filesystems. Think of it as a layer of abstraction that makes managing storage much easier and more flexible than traditional partitioning.

### Why Use LVM?

**Traditional Partitioning Problems:**
- Once you create a partition, resizing it is difficult and risky
- You're limited by physical disk boundaries
- Moving data between disks requires backup and restore
- You can't easily combine multiple disks into one large volume

**LVM Solutions:**
- Resize volumes easily (grow or shrink) while the system is running
- Combine multiple physical disks into one large pool of storage
- Move data between disks without downtime
- Create snapshots for backups
- Add more storage space without reformatting

### LVM Architecture - The Three Layers

```
┌─────────────────────────────────────────┐
│  Filesystems (ext4, xfs, etc.)          │  ← What users see
├─────────────────────────────────────────┤
│  Logical Volumes (LV)                   │  ← Virtual partitions
│  /dev/vg01/lv_data                      │
│  /dev/vg01/lv_web                       │
├─────────────────────────────────────────┤
│  Volume Groups (VG)                     │  ← Storage pool
│  vg01: combines all PVs                 │
├─────────────────────────────────────────┤
│  Physical Volumes (PV)                  │  ← Physical disks/partitions
│  /dev/sdb, /dev/sdc, /dev/sdd          │
└─────────────────────────────────────────┘
```

**1. Physical Volumes (PV)** - Your actual hard drives or partitions
   - Example: /dev/sdb, /dev/sdc1
   - These are the raw storage devices you'll use

**2. Volume Groups (VG)** - A pool of storage made from one or more PVs
   - Example: vg_data (combining /dev/sdb and /dev/sdc)
   - Think of it as a big bucket of storage space

**3. Logical Volumes (LV)** - Virtual partitions created from VG space
   - Example: lv_database, lv_webserver
   - These are what you format and mount, just like regular partitions

---

## Physical Volume Management

### What Are Physical Volumes?

Physical Volumes are your actual hard drives or partitions that have been prepared for use with LVM. Before you can use a disk with LVM, you must initialize it as a Physical Volume.

### pvcreate - Initialize a Disk for LVM

**What it does:** Prepares a disk or partition to be used with LVM by writing special metadata.

**Why use it:** This is always your first step when adding a new disk to LVM.

**Examples:**

```bash
# Initialize an entire disk for LVM
pvcreate /dev/sdb
# Output: Physical volume "/dev/sdb" successfully created.

# Initialize a partition (you must create the partition first with fdisk/parted)
pvcreate /dev/sdc1

# Initialize multiple disks at once
pvcreate /dev/sdb /dev/sdc /dev/sdd
```

**Real-world scenario:**
You've just added a new 500GB disk (/dev/sdb) to your server. Before you can use it with LVM:
```bash
pvcreate /dev/sdb
```
Now /dev/sdb is ready to be added to a Volume Group.

### pvs - Quick Overview of Physical Volumes

**What it does:** Shows a simple summary of all Physical Volumes on your system.

**Why use it:** Quick check to see what PVs exist, their size, and which VG they belong to.

**Example:**

```bash
pvs
```

**Output:**
```
PV         VG     Fmt  Attr PSize   PFree
/dev/sdb   vg01   lvm2 a--  100.00g 50.00g
/dev/sdc   vg01   lvm2 a--  200.00g 200.00g
/dev/sdd   vg02   lvm2 a--  500.00g 100.00g
```

**What this tells you:**
- /dev/sdb and /dev/sdc are both part of vg01
- /dev/sdb has 50GB free out of 100GB total
- /dev/sdc is completely unused (200GB free out of 200GB)

### pvdisplay - Detailed Information About Physical Volumes

**What it does:** Shows detailed information about one or all Physical Volumes.

**Why use it:** When you need detailed information like UUID, exact sizes, or troubleshooting.

**Examples:**

```bash
# Show details for all PVs
pvdisplay

# Show details for a specific PV
pvdisplay /dev/sdb
```

**Output example:**
```
  --- Physical volume ---
  PV Name               /dev/sdb
  VG Name               vg01
  PV Size               100.00 GiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              25599
  Free PE               12799
  Allocated PE          12800
  PV UUID               abc123-def456-...
```

**What this means:**
- This PV is part of vg01
- Total size is 100GB
- About half is used (12800 PE allocated out of 25599 total)
- PE (Physical Extent) is the smallest unit of space LVM uses (4MB blocks)

### pvmove - Move Data Off a Physical Volume

**What it does:** Moves all data from one Physical Volume to other PVs in the same Volume Group.

**Why use it:** You want to remove a disk from your system (maybe it's failing, or you're upgrading).

**Example:**

```bash
# Move all data from /dev/sdb to other disks in the VG
pvmove /dev/sdb

# Move data from /dev/sdb to a specific disk
pvmove /dev/sdb /dev/sdc
```

**Real-world scenario:**
Your /dev/sdb disk is showing SMART errors and needs to be replaced:

```bash
# Step 1: Move all data off the failing disk
# (This can take hours for large disks!)
pvmove /dev/sdb

# Step 2: Remove it from the VG
vgreduce vg01 /dev/sdb

# Step 3: Remove PV label
pvremove /dev/sdb

# Step 4: Physically replace the disk
# Step 5: Add the new disk
pvcreate /dev/sdb
vgextend vg01 /dev/sdb
```

### pvremove - Remove Physical Volume

**What it does:** Removes the LVM label from a disk, making it no longer a Physical Volume.

**Why use it:** Clean up after removing a disk from LVM, or reusing a disk for something else.

**Example:**

```bash
# Remove PV label (must not be in use!)
pvremove /dev/sdb
```

**Important:** The PV must not be part of any VG. Remove it from the VG first with `vgreduce`.

---

## Volume Group Management

### What Are Volume Groups?

A Volume Group is a storage pool created from one or more Physical Volumes. Think of it as combining multiple hard drives into one big pool of storage that you can then divide up however you want.

### vgcreate - Create a New Volume Group

**What it does:** Creates a new storage pool from one or more Physical Volumes.

**Why use it:** This is how you combine disks into a unified storage pool.

**Examples:**

```bash
# Create a VG from a single PV
vgcreate vg_data /dev/sdb

# Create a VG from multiple PVs (combining 3 disks into one pool)
vgcreate vg_data /dev/sdb /dev/sdc /dev/sdd

# Create VG with a specific extent size (16MB instead of default 4MB)
# Larger extents = slightly less overhead for very large volumes
vgcreate -s 16M vg_bigdata /dev/sdb /dev/sdc
```

**Real-world scenario:**
You have three 1TB disks and want to create one large storage pool:

```bash
# Step 1: Initialize all disks
pvcreate /dev/sdb /dev/sdc /dev/sdd

# Step 2: Create VG combining all three (total ~3TB)
vgcreate vg_storage /dev/sdb /dev/sdc /dev/sdd

# Step 3: Verify
vgs vg_storage
# Shows: vg_storage with ~3TB total size
```

### vgs - Quick Overview of Volume Groups

**What it does:** Shows a summary of all Volume Groups.

**Why use it:** Quick check of VG size, free space, and how many PVs/LVs they contain.

**Example:**

```bash
vgs
```

**Output:**
```
VG          #PV #LV #SN Attr   VSize   VFree
vg_data       2   3   0 wz--n- 300.00g  50.00g
vg_backup     1   1   0 wz--n- 500.00g 400.00g
```

**What this tells you:**
- vg_data has 2 PVs, 3 LVs, 300GB total, 50GB free
- vg_backup has 1 PV, 1 LV, 500GB total, 400GB free

### vgdisplay - Detailed Volume Group Information

**What it does:** Shows detailed information about Volume Groups.

**Why use it:** Get complete details including UUIDs, extent information, and which LVs exist.

**Example:**

```bash
# Show all VGs in detail
vgdisplay

# Show specific VG
vgdisplay vg_data
```

**Output example:**
```
  --- Volume group ---
  VG Name               vg_data
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  15
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               299.99 GiB
  PE Size               4.00 MiB
  Total PE              76798
  Alloc PE / Size       64000 / 250.00 GiB
  Free  PE / Size       12798 / 49.99 GiB
  VG UUID               xyz789-abc123-...
```

### vgextend - Add Storage to a Volume Group

**What it does:** Adds a Physical Volume to an existing Volume Group, increasing available space.

**Why use it:** When you need more space and add a new disk to the system.

**Example:**

```bash
# Add a new disk to existing VG
pvcreate /dev/sdd
vgextend vg_data /dev/sdd
```

**Real-world scenario:**
Your vg_data is running low on space. You add a new 500GB disk:

```bash
# Step 1: Initialize new disk
pvcreate /dev/sdd
# Output: Physical volume "/dev/sdd" successfully created.

# Step 2: Add to existing VG
vgextend vg_data /dev/sdd
# Output: Volume group "vg_data" successfully extended

# Step 3: Verify new space available
vgs vg_data
# VSize will now show 500GB more space
```

### vgreduce - Remove a Disk from Volume Group

**What it does:** Removes a Physical Volume from a Volume Group.

**Why use it:** Before removing a disk or decommissioning storage.

**Example:**

```bash
# First, move data off the disk
pvmove /dev/sdd

# Then remove it from the VG
vgreduce vg_data /dev/sdd

# Finally, remove PV label
pvremove /dev/sdd
```

**Real-world scenario:**
You want to remove /dev/sdd from vg_data:

```bash
# Check what's on it first
pvs /dev/sdd

# Move any data to other disks in the VG
pvmove /dev/sdd
# This may take a while...

# Remove from VG
vgreduce vg_data /dev/sdd
# Output: Volume group "vg_data" successfully reduced

# Clean up
pvremove /dev/sdd
```

### vgremove - Delete a Volume Group

**What it does:** Completely removes a Volume Group.

**Why use it:** Cleaning up or starting over with storage configuration.

**Example:**

```bash
# Remove all LVs first
lvremove /dev/vg_data/lv_web
lvremove /dev/vg_data/lv_db

# Then remove the VG
vgremove vg_data
```

**Warning:** This is destructive! All Logical Volumes must be removed first.

---

## Logical Volume Management

### What Are Logical Volumes?

Logical Volumes are virtual partitions created from Volume Group space. They're what you actually format with filesystems and mount. Unlike traditional partitions, they can be easily resized, moved, and snapshotted.

### lvcreate - Create a Logical Volume

**What it does:** Creates a new Logical Volume from available space in a Volume Group.

**Why use it:** This creates the "partition" you'll format and use for storing data.

**Examples:**

```bash
# Create a 20GB logical volume named "lv_web"
lvcreate -L 20G -n lv_web vg_data

# Create an LV using 50% of the VG
lvcreate -l 50%VG -n lv_database vg_data

# Create an LV using ALL free space
lvcreate -l 100%FREE -n lv_backup vg_data

# Create an LV with exactly 5000 extents (size depends on PE size)
lvcreate -l 5000 -n lv_app vg_data
```

**Real-world scenario - Setting up a web server:**

```bash
# You have vg_data with 500GB free space
# Create volumes for different purposes:

# 50GB for web files
lvcreate -L 50G -n lv_web vg_data

# 100GB for database
lvcreate -L 100G -n lv_database vg_data

# 30GB for logs
lvcreate -L 30G -n lv_logs vg_data

# Use remaining space for backups
lvcreate -l 100%FREE -n lv_backup vg_data

# Verify
lvs vg_data
```

**Complete workflow - From disk to mounted filesystem:**

```bash
# 1. Create the LV
lvcreate -L 50G -n lv_webapp vg_data

# 2. Create a filesystem on it
mkfs.ext4 /dev/vg_data/lv_webapp

# 3. Create mount point
mkdir /var/www

# 4. Mount it
mount /dev/vg_data/lv_webapp /var/www

# 5. Make it permanent in /etc/fstab
echo "/dev/vg_data/lv_webapp /var/www ext4 defaults 0 2" >> /etc/fstab

# 6. Verify
df -h /var/www
```

### lvs - Quick Overview of Logical Volumes

**What it does:** Shows a summary of all Logical Volumes.

**Why use it:** Quick check of LV sizes, which VG they're in, and their status.

**Example:**

```bash
lvs
```

**Output:**
```
LV          VG       Attr       LSize   Pool Origin Data%
lv_database vg_data  -wi-ao---- 100.00g
lv_web      vg_data  -wi-ao----  50.00g
lv_backup   vg_data  -wi-a----- 200.00g
```

**What the attributes mean:**
- `w` = writable
- `i` = inherited allocation policy
- `a` = active (usable)
- `o` = open (currently mounted)

### lvdisplay - Detailed Logical Volume Information

**What it does:** Shows detailed information about Logical Volumes.

**Why use it:** Get full details including device paths, segments, and exact sizes.

**Example:**

```bash
# Show all LVs
lvdisplay

# Show specific LV
lvdisplay /dev/vg_data/lv_web
```

**Output example:**
```
  --- Logical volume ---
  LV Path                /dev/vg_data/lv_web
  LV Name                lv_web
  VG Name                vg_data
  LV UUID                mno789-pqr012-...
  LV Write Access        read/write
  LV Creation host, time server01, 2024-10-28 10:30:15
  LV Status              available
  # open                 1
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  Block device           253:0
```

### lvextend - Increase Logical Volume Size

**What it does:** Makes a Logical Volume larger by allocating more space from the Volume Group.

**Why use it:** When you're running out of space on a filesystem and need more room.

**Examples:**

```bash
# Add 10GB to an LV
lvextend -L +10G /dev/vg_data/lv_web

# Extend to a total of 100GB
lvextend -L 100G /dev/vg_data/lv_web

# Use all remaining free space in VG
lvextend -l +100%FREE /dev/vg_data/lv_web

# Extend AND resize the filesystem in one command (convenient!)
lvextend -L +10G -r /dev/vg_data/lv_web
```

**Real-world scenario - Running out of space:**

You're getting warnings that /var/www is 90% full:

```bash
# Check current situation
df -h /var/www
# Shows: 45GB used out of 50GB (90% full)

lvs /dev/vg_data/lv_web
# Shows: 50GB LV

vgs vg_data
# Shows: 200GB free in VG

# Solution: Add 30GB more
lvextend -L +30G /dev/vg_data/lv_web
# Output: Size of logical volume vg_data/lv_web changed from 50.00 GiB to 80.00 GiB

# Resize the filesystem to use new space
resize2fs /dev/vg_data/lv_web
# Output: The filesystem is now 20971520 blocks long

# Verify
df -h /var/www
# Shows: 45GB used out of 80GB (56% full)
```

**Important:** After extending the LV, you must resize the filesystem:
- **ext4/ext3/ext2:** `resize2fs /dev/vg_data/lv_name`
- **xfs:** `xfs_growfs /mount/point`
- **Or use `-r` flag** with lvextend to do both automatically

### lvreduce - Decrease Logical Volume Size

**What it does:** Makes a Logical Volume smaller, freeing space back to the Volume Group.

**Why use it:** When you over-allocated space and want to reclaim it for other uses.

**WARNING:** This is dangerous! Always backup data first!

**Example:**

```bash
# CRITICAL: Shrink filesystem FIRST, then LV
# If you shrink the LV first, you'll lose data!

# For ext4:
# Step 1: Unmount (required for shrinking)
umount /mnt/data

# Step 2: Check filesystem
e2fsck -f /dev/vg_data/lv_data

# Step 3: Shrink filesystem to 30GB
resize2fs /dev/vg_data/lv_data 30G

# Step 4: Shrink LV to match
lvreduce -L 30G /dev/vg_data/lv_data
# It will ask for confirmation - type 'y'

# Step 5: Remount
mount /dev/vg_data/lv_data /mnt/data
```

**Real-world scenario:**
You allocated 100GB for /opt/app but only use 25GB:

```bash
# Check usage
df -h /opt/app
# Shows: 25GB used out of 100GB

# Back up data first! (Just in case)
tar czf /backup/app-backup.tar.gz /opt/app

# Unmount
umount /opt/app

# Check and repair filesystem
e2fsck -f /dev/vg_data/lv_app

# Shrink filesystem to 40GB (leaving room for growth)
resize2fs /dev/vg_data/lv_app 40G

# Shrink LV to match
lvreduce -L 40G /dev/vg_data/lv_app

# Remount
mount /dev/vg_data/lv_app /opt/app

# Verify
df -h /opt/app
# Now shows: 25GB used out of 40GB

# The freed 60GB is now available in vg_data for other uses
vgs vg_data
```

**Note:** XFS filesystems **cannot be shrunk** - only grown! Plan sizes carefully.

### lvremove - Delete a Logical Volume

**What it does:** Completely removes a Logical Volume and frees its space back to the VG.

**Why use it:** Cleaning up unused volumes or removing test environments.

**Example:**

```bash
# Unmount first
umount /mnt/old

# Remove the LV
lvremove /dev/vg_data/lv_old
# Asks: "Do you really want to remove...?" type 'y'

# Or force removal without confirmation (dangerous!)
lvremove -f /dev/vg_data/lv_old
```

---

## LVM Snapshots

### What Are Snapshots?

A snapshot is a point-in-time copy of a Logical Volume. It doesn't copy all the data immediately; instead, it uses copy-on-write technology:
- When you create a snapshot, it initially uses very little space
- As the original LV changes, the snapshot stores the old data
- The snapshot shows how the LV looked at the moment you created it

### Why Use Snapshots?

**Perfect for:**
- Backing up a live database without stopping it
- Testing changes safely (take snapshot, test, revert if needed)
- Creating consistent backups of busy filesystems

**Example Use Case:**
You need to backup a database that runs 24/7. You can't stop it, but you need a consistent backup:
1. Take a snapshot (takes seconds)
2. The snapshot is a frozen point-in-time view
3. Back up from the snapshot (take hours if needed)
4. Original database keeps running and changing
5. Delete snapshot when backup complete

### Creating Snapshots

```bash
# Create a 10GB snapshot of lv_database
lvcreate -L 10G -s -n lv_database_snap /dev/vg_data/lv_database

# The size (10GB) is for storing changes, not the whole database
# Size needed depends on how much changes during snapshot lifetime
```

**Real-world scenario - Database backup:**

```bash
# Your database LV is 100GB, actively being used
# Create snapshot (only needs space for changes during backup)
lvcreate -L 20G -s -n lv_db_backup /dev/vg_data/lv_database
# Output: Logical volume "lv_db_backup" created

# Mount the snapshot read-only
mkdir /mnt/snap
mount -o ro /dev/vg_data/lv_db_backup /mnt/snap

# Backup from snapshot (original database keeps running!)
tar czf /backup/database_$(date +%Y%m%d).tar.gz -C /mnt/snap .

# Cleanup
umount /mnt/snap
lvremove /dev/vg_data/lv_db_backup
```

### Monitoring Snapshot Usage

**What to watch:** Snapshots need space to store changes. If they fill up, they become invalid!

```bash
# Check snapshot usage
lvs -a -o +snap_percent

# Output shows percentage used:
# LV                VG      ...  Snap%
# lv_database_snap  vg_data ...  15.23
```

If it reaches 100%, the snapshot becomes invalid and useless. Solution: make snapshot size larger.

### Extending a Snapshot

```bash
# If snapshot is getting full, extend it
lvextend -L +5G /dev/vg_data/lv_database_snap
```

### Reverting with Snapshots (Rollback)

**What it does:** Merges the snapshot back to the original, reverting all changes.

**When to use:** You tested something, it broke, and you want to undo everything.

**Example:**

```bash
# Before major system update:
# Take snapshot
lvcreate -L 20G -s -n lv_root_snap /dev/vg_system/lv_root

# Perform risky operation (install updates, etc.)
dnf update -y

# If something breaks, revert:
umount /
lvconvert --merge /dev/vg_system/lv_root_snap
# Reboot required to complete merge
reboot

# System comes back to pre-update state!
```

**Warning:** Merging destroys the snapshot and reverts the original LV. Make sure this is what you want!

---

## Complete Real-World Examples

### Example 1: Setting Up LVM from Scratch

**Scenario:** New server with three empty disks. Set up LVM for flexible storage.

```bash
# Step 1: Initialize physical disks
pvcreate /dev/sdb /dev/sdc /dev/sdd
# Output: Physical volume "/dev/sdb" successfully created.
#         Physical volume "/dev/sdc" successfully created.
#         Physical volume "/dev/sdd" successfully created.

# Step 2: Create volume group combining all disks
vgcreate vg_data /dev/sdb /dev/sdc /dev/sdd
# Output: Volume group "vg_data" successfully created

# Step 3: Check available space
vgs vg_data
# Shows total combined space (e.g., 1.5TB)

# Step 4: Create logical volumes for different purposes
lvcreate -L 200G -n lv_database vg_data
lvcreate -L 100G -n lv_webapp vg_data
lvcreate -L 50G -n lv_logs vg_data
lvcreate -l 100%FREE -n lv_backup vg_data

# Step 5: Create filesystems
mkfs.ext4 /dev/vg_data/lv_database
mkfs.xfs /dev/vg_data/lv_webapp
mkfs.ext4 /dev/vg_data/lv_logs
mkfs.ext4 /dev/vg_data/lv_backup

# Step 6: Create mount points
mkdir -p /data/database
mkdir -p /var/www
mkdir -p /var/log/apps
mkdir -p /backup

# Step 7: Mount filesystems
mount /dev/vg_data/lv_database /data/database
mount /dev/vg_data/lv_webapp /var/www
mount /dev/vg_data/lv_logs /var/log/apps
mount /dev/vg_data/lv_backup /backup

# Step 8: Make permanent in /etc/fstab
cat >> /etc/fstab << EOF
/dev/vg_data/lv_database /data/database ext4 defaults 0 2
/dev/vg_data/lv_webapp /var/www xfs defaults 0 2
/dev/vg_data/lv_logs /var/log/apps ext4 defaults 0 2
/dev/vg_data/lv_backup /backup ext4 defaults 0 2
EOF

# Step 9: Verify everything
df -h
lvs
```

### Example 2: Expanding Storage When Running Out of Space

**Scenario:** /var/www is 95% full and you need more space immediately.

```bash
# Step 1: Check current situation
df -h /var/www
# Filesystem                Size  Used Avail Use% Mounted on
# /dev/vg_data/lv_webapp    50G   48G   2G  96% /var/www

# Step 2: Check if VG has free space
vgs vg_data
# VG       #PV #LV #SN Attr   VSize  VFree
# vg_data    3   4   0 wz--n- 1.50t  500.00g

# Good! We have 500GB free

# Step 3: Extend the LV by 50GB
lvextend -L +50G /dev/vg_data/lv_webapp
# Output: Size of logical volume vg_data/lv_webapp changed from 50.00 GiB to 100.00 GiB

# Step 4: Resize the filesystem (xfs in this case)
xfs_growfs /var/www
# Output: data blocks changed from 13107200 to 26214400

# OR for ext4, use:
# resize2fs /dev/vg_data/lv_webapp

# Step 5: Verify
df -h /var/www
# Filesystem                Size  Used Avail Use% Mounted on
# /dev/vg_data/lv_webapp   100G   48G   52G  48% /var/www

# Problem solved! No downtime needed.
```

### Example 3: Adding a New Disk to Existing Setup

**Scenario:** Server is running low on storage across all volumes. Add new 1TB disk.

```bash
# Step 1: Initialize new disk
pvcreate /dev/sde
# Output: Physical volume "/dev/sde" successfully created.

# Step 2: Add to existing volume group
vgextend vg_data /dev/sde
# Output: Volume group "vg_data" successfully extended

# Step 3: Verify new space available
vgs vg_data
# Now shows 1TB more space in VFree

# Step 4: Extend volumes that need space
# Database needs more space
lvextend -L +200G /dev/vg_data/lv_database
resize2fs /dev/vg_data/lv_database

# Backup needs more space
lvextend -L +300G /dev/vg_data/lv_backup
resize2fs /dev/vg_data/lv_backup

# Step 5: Verify
df -h
lvs vg_data
```

### Example 4: Safe Database Maintenance with Snapshots

**Scenario:** Need to perform risky database migrations. Want ability to rollback.

```bash
# Step 1: Stop database (if possible) or at least sync
systemctl stop postgresql

# Step 2: Create snapshot (20GB for changes)
lvcreate -L 20G -s -n lv_db_premigration /dev/vg_data/lv_database
# Output: Logical volume "lv_db_premigration" created

# Step 3: Start database and perform migrations
systemctl start postgresql
./run-database-migration.sh

# Step 4a: If migration successful, remove snapshot
lvremove /dev/vg_data/lv_db_premigration

# Step 4b: If migration failed, rollback
systemctl stop postgresql
umount /data/database
lvconvert --merge /dev/vg_data/lv_db_premigration
# Merge will happen on next mount/reboot
mount /dev/vg_data/lv_database /data/database
systemctl start postgresql
# Database is now back to pre-migration state!
```

### Example 5: Replacing a Failing Disk

**Scenario:** SMART errors on /dev/sdc. Need to replace without downtime.

```bash
# Step 1: Add new replacement disk
pvcreate /dev/sdf
vgextend vg_data /dev/sdf

# Step 2: Move data from failing disk to new disk
# This can take hours! System stays online.
pvmove /dev/sdc /dev/sdf
# Shows progress: /dev/sdc: Moved: 45.2%

# Step 3: After pvmove completes, remove old disk from VG
vgreduce vg_data /dev/sdc

# Step 4: Remove PV label
pvremove /dev/sdc

# Step 5: Physically remove /dev/sdc
# No data loss, no downtime!
```

---

## Troubleshooting Common Issues

### Issue 1: "Cannot create LV - insufficient space"

**Problem:** Trying to create LV but get error about insufficient space.

```bash
# Check VG free space
vgs vg_data

# If VFree is 0, you need to either:
# a) Add more disks
pvcreate /dev/sde
vgextend vg_data /dev/sde

# b) Remove or shrink other LVs to free space
lvremove /dev/vg_data/lv_unused
# or
lvreduce -L 50G /dev/vg_data/lv_oversized
```

### Issue 2: "Device or resource busy" when removing LV

**Problem:** Can't remove an LV because it's in use.

```bash
# Find what's using it
lsof /dev/vg_data/lv_name
# or
fuser -m /mount/point

# Unmount it
umount /mount/point

# Try again
lvremove /dev/vg_data/lv_name
```

### Issue 3: Snapshot became invalid

**Problem:** Snapshot shows "Snapshot status: INVALID"

**Cause:** Snapshot ran out of space to store changes.

```bash
# Check snapshot usage
lvs -a -o +snap_percent

# If near 100%, extend it quickly
lvextend -L +10G /dev/vg_data/lv_snap

# For future: create larger snapshots initially
```

### Issue 4: Cannot activate LV

**Problem:** LV won't activate after reboot.

```bash
# Scan for LVM volumes
pvscan
vgscan
lvscan

# Activate all
vgchange -ay

# Activate specific VG
vgchange -ay vg_data
```

---

## Best Practices

**1. Plan Before Creating**
- Think about future growth
- Leave free space in VG (20-30%) for flexibility
- Use logical naming (lv_database, not lv1)

**2. Regular Monitoring**
- Check space regularly: `df -h` and `lvs`
- Monitor VG free space: `vgs`
- Set up alerts at 80% full

**3. Backup Before Changes**
- Always backup before shrinking
- Take snapshots before risky operations
- Document your LVM layout

**4. Extent Size**
- Default 4MB is fine for most cases
- Use larger (16MB+) for very large volumes (multiple TB)

**5. Snapshot Management**
- Size snapshots appropriately (10-30% of original)
- Don't keep snapshots long-term (performance impact)
- Monitor snapshot usage

**6. Documentation**
- Keep diagrams of your LVM setup
- Document which LVs contain what data
- Note any special configurations

---

## Quick Command Reference

```bash
# === Creating LVM from scratch ===
pvcreate /dev/sdb                    # Initialize disk
vgcreate vg_name /dev/sdb           # Create volume group
lvcreate -L 50G -n lv_name vg_name  # Create logical volume
mkfs.ext4 /dev/vg_name/lv_name      # Format with filesystem
mount /dev/vg_name/lv_name /mnt     # Mount it

# === Viewing status ===
pvs                                  # List physical volumes
vgs                                  # List volume groups
lvs                                  # List logical volumes

# === Expanding storage ===
lvextend -L +10G /dev/vg/lv         # Add 10GB to LV
resize2fs /dev/vg/lv                # Grow ext4 filesystem
xfs_growfs /mount/point             # Grow XFS filesystem

# === Snapshots ===
lvcreate -L 10G -s -n snap /dev/vg/lv  # Create snapshot
lvremove /dev/vg/snap                   # Remove snapshot
lvconvert --merge /dev/vg/snap          # Revert to snapshot

# === Adding storage ===
pvcreate /dev/sdc                    # Initialize new disk
vgextend vg_name /dev/sdc           # Add to volume group

# === Removing storage ===
pvmove /dev/sdc                      # Move data off disk
vgreduce vg_name /dev/sdc           # Remove from VG
pvremove /dev/sdc                    # Clean up
```
