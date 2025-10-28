# Configure and Manage LVM Storage

## Overview
LVM (Logical Volume Manager) provides flexible disk management by abstracting physical storage into logical volumes. It consists of three layers: Physical Volumes (PV), Volume Groups (VG), and Logical Volumes (LV).

## Architecture
```
Physical Disks → Physical Volumes (PV) → Volume Groups (VG) → Logical Volumes (LV) → Filesystems
```

---

## Physical Volume Management

### pvs - Display Physical Volume Information
```bash
pvs
```
**Common Options:**

- `-o` : Specify output columns
- `-v` : Verbose output
- `--segments` : Display segment information

**Use Cases:**

- Quick overview of all PVs
- Check PV utilization

### pvdisplay - Detailed Physical Volume Information
```bash
pvdisplay [PV_path]
pvdisplay /dev/sdb1
```
**Common Options:**

- `-v` : Verbose output
- `-m` : Display mapping of physical extents
- `-s` : Short format

**Use Cases:**

- Detailed PV examination
- View PV UUID and size
- Check which VG a PV belongs to

### pvcreate - Create Physical Volume
```bash
pvcreate /dev/sdb
pvcreate /dev/sdb1 /dev/sdc1
```
**Common Options:**

- `-f` : Force creation
- `--metadatasize` : Set metadata area size
- `-y` : Automatic yes to prompts

**Use Cases:**

- Initialize disk for LVM use
- Prepare partitions for VG
- Convert existing disk to PV

**Example:**
```bash
# Create PV on entire disk
pvcreate /dev/sdb

# Create PV on multiple partitions
pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1
```

### pvremove - Remove Physical Volume
```bash
pvremove /dev/sdb
```
**Common Options:**

- `-f` : Force removal
- `-y` : Automatic yes

**Use Cases:**

- Decommission disk from LVM
- Remove unused PV

### pvmove - Move Data Between Physical Volumes
```bash
pvmove /dev/sdb1 /dev/sdc1
```
**Common Options:**

- `-i` : Interval for progress updates
- `-n` : Only move specified logical volume
- `--abort` : Abort in-progress move

**Use Cases:**

- Migrate data before disk removal
- Balance data across PVs
- Move data off failing disk

**Example:**
```bash
# Move all data from sdb1 to sdc1
pvmove /dev/sdb1 /dev/sdc1

# Move specific LV only
pvmove -n vg01/lv_data /dev/sdb1 /dev/sdc1
```

### pvresize - Resize Physical Volume
```bash
pvresize /dev/sdb1
```
**Common Options:**

- `--setphysicalvolumesize` : Set specific size

**Use Cases:**

- Expand PV after partition resize
- Shrink PV before partition reduction

---

## Volume Group Management

### vgs - Display Volume Group Information
```bash
vgs
```
**Common Options:**

- `-o` : Specify output columns
- `-v` : Verbose output
- `--units` : Set display units (k/m/g/t)

**Use Cases:**

- Quick VG overview
- Check available space

### vgdisplay - Detailed Volume Group Information
```bash
vgdisplay [VG_name]
vgdisplay vg01
```
**Common Options:**

- `-v` : Verbose output (shows LVs and PVs)
- `-s` : Short format
- `-A` : Show active VGs only

**Use Cases:**

- Detailed VG examination
- View VG size and free space
- List all LVs in VG

### vgcreate - Create Volume Group
```bash
vgcreate vg01 /dev/sdb1 /dev/sdc1
```
**Common Options:**

- `-s` : Set physical extent size
- `-l` : Set maximum logical volumes
- `-p` : Set maximum physical volumes

**Use Cases:**

- Create new storage pool
- Group multiple PVs

**Example:**
```bash
# Create VG with default settings
vgcreate vg01 /dev/sdb1

# Create VG with 8MB extent size
vgcreate -s 8M vg01 /dev/sdb1 /dev/sdc1

# Create VG with specific extent size
vgcreate -s 16M vg_data /dev/sd{b,c,d}1
```

### vgextend - Add Physical Volume to Volume Group
```bash
vgextend vg01 /dev/sdd1
```
**Common Options:**

- `-A` : Set autobackup
- `-t` : Test mode

**Use Cases:**

- Expand VG capacity
- Add new disk to existing VG

**Example:**
```bash
# First create PV, then extend VG
pvcreate /dev/sdd1
vgextend vg01 /dev/sdd1
```

### vgreduce - Remove Physical Volume from Volume Group
```bash
vgreduce vg01 /dev/sdd1
```
**Common Options:**

- `--removemissing` : Remove missing PVs
- `-a` : Remove all empty PVs
- `-f` : Force removal

**Use Cases:**

- Remove unused PV from VG
- Remove failed disk

**Example:**
```bash
# Move data off PV first, then reduce
pvmove /dev/sdd1
vgreduce vg01 /dev/sdd1

# Remove missing/failed PVs
vgreduce --removemissing vg01
```

### vgremove - Remove Volume Group
```bash
vgremove vg01
```
**Common Options:**

- `-f` : Force removal

**Use Cases:**

- Delete entire VG
- Clean up unused storage pool

### vgrename - Rename Volume Group
```bash
vgrename vg01 vg_data
vgrename /dev/vg01 /dev/vg_data
```
**Use Cases:**

- Change VG name for clarity
- Standardize naming

### vgchange - Change Volume Group Attributes
```bash
vgchange -a y vg01
```
**Common Options:**

- `-a y|n` : Activate/deactivate VG
- `-l` : Set maximum logical volumes
- `--refresh` : Refresh metadata

**Use Cases:**

- Activate/deactivate VG
- Enable VG at boot
- Refresh VG state

---

## Logical Volume Management

### lvs - Display Logical Volume Information
```bash
lvs
```
**Common Options:**

- `-o` : Specify output columns
- `-a` : Show all LVs including hidden
- `--units` : Set display units

**Use Cases:**

- Quick LV overview
- Check LV size and attributes

### lvdisplay - Detailed Logical Volume Information
```bash
lvdisplay [VG_name/LV_name]
lvdisplay /dev/vg01/lv_data
```
**Common Options:**

- `-v` : Verbose output
- `-m` : Display mapping
- `--maps` : Show extent mappings

**Use Cases:**

- Detailed LV examination
- View LV path and size
- Check LV segments

### lvcreate - Create Logical Volume
```bash
lvcreate -L 10G -n lv_data vg01
```
**Common Options:**

- `-L` : Size in bytes (K/M/G/T)
- `-l` : Size in extents or percentage
- `-n` : Name of logical volume
- `-s` : Create snapshot
- `--type` : Specify LV type (linear, striped, mirror, raid)

**Use Cases:**

- Create storage volumes
- Create snapshots for backups
- Set up mirrored volumes

**Examples:**
```bash
# Create 10GB LV
lvcreate -L 10G -n lv_data vg01

# Create LV using 50% of VG
lvcreate -l 50%VG -n lv_data vg01

# Create LV using 100% of free space
lvcreate -l 100%FREE -n lv_data vg01

# Create LV with specific extent count
lvcreate -l 1000 -n lv_data vg01

# Create snapshot (5GB snapshot of lv_data)
lvcreate -L 5G -s -n lv_data_snap /dev/vg01/lv_data

# Create striped LV across 2 PVs
lvcreate -L 20G -i 2 -n lv_striped vg01

# Create mirrored LV
lvcreate -L 10G -m1 -n lv_mirror vg01
```

### lvextend - Extend Logical Volume
```bash
lvextend -L +5G /dev/vg01/lv_data
```
**Common Options:**

- `-L` : New size or size to add (+)
- `-l` : Extents to add
- `-r` : Resize filesystem automatically

**Use Cases:**

- Expand volume when space needed
- Resize filesystem in one command

**Examples:**
```bash
# Add 5GB to LV
lvextend -L +5G /dev/vg01/lv_data

# Extend to 20GB total
lvextend -L 20G /dev/vg01/lv_data

# Extend to 100% of VG free space
lvextend -l +100%FREE /dev/vg01/lv_data

# Extend and resize filesystem
lvextend -L +5G -r /dev/vg01/lv_data
```

### lvreduce - Reduce Logical Volume
```bash
lvreduce -L -5G /dev/vg01/lv_data
```
**Common Options:**

- `-L` : New size or size to reduce (-)
- `-l` : Extents to reduce
- `-r` : Resize filesystem first
- `-f` : Force without confirmation

**Use Cases:**

- Reclaim unused space
- Shrink oversized volumes

**Examples:**
```bash
# ALWAYS shrink filesystem BEFORE reducing LV
resize2fs /dev/vg01/lv_data 10G
lvreduce -L 10G /dev/vg01/lv_data

# Reduce by 5GB (with filesystem resize)
lvreduce -L -5G -r /dev/vg01/lv_data
```

**WARNING:** Always backup data before reducing. Shrink filesystem BEFORE reducing LV size.

### lvresize - Resize Logical Volume
```bash
lvresize -L 15G /dev/vg01/lv_data
```
**Common Options:**

- `-L` : Set absolute size
- `-l` : Size in extents
- `-r` : Resize filesystem automatically

**Use Cases:**

- Unified command for extend/reduce
- Set exact size

**Examples:**
```bash
# Resize to exactly 15GB
lvresize -L 15G -r /dev/vg01/lv_data

# Resize to 80% of VG
lvresize -l 80%VG -r /dev/vg01/lv_data
```

### lvremove - Remove Logical Volume
```bash
lvremove /dev/vg01/lv_data
```
**Common Options:**

- `-f` : Force removal

**Use Cases:**

- Delete unused LV
- Clean up test volumes

**Example:**
```bash
# Unmount first, then remove
umount /mnt/data
lvremove /dev/vg01/lv_data
```

### lvrename - Rename Logical Volume
```bash
lvrename vg01 lv_old lv_new
lvrename /dev/vg01/lv_old /dev/vg01/lv_new
```
**Use Cases:**

- Change LV name for clarity
- Standardize naming

### lvchange - Change Logical Volume Attributes
```bash
lvchange -a y /dev/vg01/lv_data
```
**Common Options:**

- `-a y|n` : Activate/deactivate LV
- `-p r|rw` : Change permissions
- `--refresh` : Refresh LV

**Use Cases:**

- Activate/deactivate volumes
- Change read-only status

---

## LVM Snapshots

### Create Snapshot
```bash
lvcreate -L 5G -s -n lv_data_snap /dev/vg01/lv_data
```
**Use Cases:**

- Backup consistent state
- Test changes safely
- Rollback capability

### Merge Snapshot (Rollback)
```bash
lvconvert --merge /dev/vg01/lv_data_snap
```
**Note:** LV must be inactive for merge. System will merge on next activation.

**Example:**
```bash
# Unmount, merge, and remount
umount /mnt/data
lvconvert --merge /dev/vg01/lv_data_snap
mount /dev/vg01/lv_data /mnt/data
```

---

## Complete Workflow Examples

### Create Complete LVM Setup
```bash
# 1. Create physical volumes
pvcreate /dev/sdb /dev/sdc

# 2. Create volume group
vgcreate vg_data /dev/sdb /dev/sdc

# 3. Create logical volume
lvcreate -L 20G -n lv_webapp vg_data

# 4. Create filesystem
mkfs.ext4 /dev/vg_data/lv_webapp

# 5. Mount
mkdir /var/webapp
mount /dev/vg_data/lv_webapp /var/webapp

# 6. Add to /etc/fstab
echo "/dev/vg_data/lv_webapp /var/webapp ext4 defaults 0 2" >> /etc/fstab
```

### Extend LVM Volume
```bash
# 1. Check current size
df -h /var/webapp
lvdisplay /dev/vg_data/lv_webapp

# 2. Check VG free space
vgs vg_data

# 3. Extend LV
lvextend -L +10G /dev/vg_data/lv_webapp

# 4. Resize filesystem (ext4)
resize2fs /dev/vg_data/lv_webapp

# 5. Verify
df -h /var/webapp
```

### Add New Disk to Existing VG
```bash
# 1. Create PV on new disk
pvcreate /dev/sdd

# 2. Extend VG
vgextend vg_data /dev/sdd

# 3. Verify
vgs vg_data
pvs
```

### Remove Disk from VG
```bash
# 1. Move data off disk
pvmove /dev/sdc

# 2. Reduce VG
vgreduce vg_data /dev/sdc

# 3. Remove PV
pvremove /dev/sdc

# 4. Verify
vgs vg_data
pvs
```

### Create and Use Snapshot for Backup
```bash
# 1. Create snapshot (10% of original size usually sufficient)
lvcreate -L 5G -s -n lv_webapp_backup /dev/vg_data/lv_webapp

# 2. Mount snapshot
mkdir /mnt/backup
mount -o ro /dev/vg_data/lv_webapp_backup /mnt/backup

# 3. Backup data
tar czf /backups/webapp_$(date +%Y%m%d).tar.gz -C /mnt/backup .

# 4. Cleanup
umount /mnt/backup
lvremove /dev/vg_data/lv_webapp_backup
```

---

## Troubleshooting

### Check LVM Components Status
```bash
# Check all components
pvs
vgs
lvs

# Detailed view
pvdisplay
vgdisplay
lvdisplay
```

### Activate Inactive Volume Group
```bash
vgchange -a y vg_data
```

### Scan for LVM Volumes
```bash
pvscan
vgscan
lvscan
```

### Repair Volume Group Metadata
```bash
vgcfgrestore -l vg_data    # List backups
vgcfgrestore vg_data       # Restore from automatic backup
```

### Handle Missing Physical Volumes
```bash
# Remove missing PVs from VG
vgreduce --removemissing vg_data

# If data is affected
vgreduce --removemissing --force vg_data
```

### Check Snapshot Usage
```bash
lvs -a -o +snap_percent
```

### Extend Snapshot That's Getting Full
```bash
lvextend -L +2G /dev/vg_data/lv_data_snap
```

---

## Quick Reference

### Size Specifications
- **K, M, G, T**: Kilobytes, Megabytes, Gigabytes, Terabytes
- **+SIZE**: Add to current size
- **-SIZE**: Subtract from current size
- **%VG**: Percentage of volume group
- **%FREE**: Percentage of free space
- **%PVS**: Percentage of physical volume space

### Common Size Examples
```bash
lvcreate -L 10G          # 10 Gigabytes
lvcreate -L 512M         # 512 Megabytes
lvcreate -l 100%FREE     # All free space
lvcreate -l 50%VG        # Half of VG
lvextend -L +5G          # Add 5GB
lvextend -l +100%FREE    # Add all free space
```
