# Storage

> Reference: [Kodekloud public notes](https://notes.kodekloud.com/docs/Linux-Foundation-Certified-System-Administrator-LFCS/Storage/Storage)

## Table of Contents

- [Storage](#storage)
  - [Table of Contents](#table-of-contents)
  - [Physical Storage Partitions](#physical-storage-partitions)
    - [Viewing Disk and Partition Information](#viewing-disk-and-partition-information)
    - [Creating Partitions with fdisk](#creating-partitions-with-fdisk)
    - [Creating Partitions with cfdisk (Interactive GUI)](#creating-partitions-with-cfdisk-interactive-gui)
    - [Creating Partitions with parted](#creating-partitions-with-parted)
    - [Partition Type Changes](#partition-type-changes)
  - [Swap Space Management](#swap-space-management)
    - [Viewing Swap Information](#viewing-swap-information)
    - [Creating Swap Partition](#creating-swap-partition)
    - [Creating Swap File](#creating-swap-file)
    - [Making Swap Persistent](#making-swap-persistent)
    - [Adjusting Swappiness](#adjusting-swappiness)
  - [Filesystems](#filesystems)
    - [Creating Filesystems](#creating-filesystems)
    - [Viewing Filesystem Information](#viewing-filesystem-information)
    - [Checking and Repairing Filesystems](#checking-and-repairing-filesystems)
    - [Tuning Filesystems](#tuning-filesystems)
  - [Mounting Filesystems](#mounting-filesystems)
    - [Manual Mounting](#manual-mounting)
    - [Persistent Mounting with /etc/fstab](#persistent-mounting-with-etcfstab)
    - [Finding UUID and Labels](#finding-uuid-and-labels)
  - [Filesystem and Mount Options](#filesystem-and-mount-options)
    - [Common Mount Options](#common-mount-options)
    - [fstab Mount Options](#fstab-mount-options)
  - [NFS - Remote Filesystems](#nfs---remote-filesystems)
    - [Installing NFS Client](#installing-nfs-client)
    - [Viewing NFS Exports](#viewing-nfs-exports)
    - [Mounting NFS Shares](#mounting-nfs-shares)
    - [Persistent NFS Mounts](#persistent-nfs-mounts)
    - [NFS Mount Options](#nfs-mount-options)
    - [NFS Server Configuration](#nfs-server-configuration)
  - [Network Block Devices (NBD)](#network-block-devices-nbd)
    - [Installing NBD](#installing-nbd)
    - [NBD Client Commands](#nbd-client-commands)
    - [Using NBD Devices](#using-nbd-devices)
    - [NBD Server Configuration](#nbd-server-configuration)
  - [LVM Storage Management](#lvm-storage-management)
    - [Physical Volume (PV) Management](#physical-volume-pv-management)
    - [Volume Group (VG) Management](#volume-group-vg-management)
    - [Logical Volume (LV) Management](#logical-volume-lv-management)
    - [Resizing Filesystems on LVM](#resizing-filesystems-on-lvm)
    - [LVM Snapshots](#lvm-snapshots)
  - [Storage Performance Monitoring](#storage-performance-monitoring)
    - [Disk Usage Commands](#disk-usage-commands)
    - [I/O Performance Monitoring](#io-performance-monitoring)
    - [Disk Performance Testing](#disk-performance-testing)
    - [System Resource Monitoring](#system-resource-monitoring)
  - [Advanced Filesystem Permissions](#advanced-filesystem-permissions)
    - [Standard Permissions](#standard-permissions)
    - [Special Permissions](#special-permissions)
    - [Access Control Lists (ACLs)](#access-control-lists-acls)
    - [File Attributes](#file-attributes)
    - [umask (Default Permissions)](#umask-default-permissions)

---

## Physical Storage Partitions

### Viewing Disk and Partition Information

```bash
# List block devices in tree format
lsblk
```

```bash
# List all disks and partitions with details
sudo fdisk -l
# Or for specific disk:
sudo fdisk -l /dev/sda
```

```bash
# View detailed partition information with fdisk
sudo fdisk --list /dev/sda
```

```bash
# View files representing partitions
ls /dev/sda*
ls -l /dev/sda1
```

```bash
# View partition information with parted
sudo parted /dev/sda print
```

### Creating Partitions with fdisk

```bash
# Open fdisk interactive partition editor
sudo fdisk /dev/sdb
```

fdisk interactive commands:
- `p` - Print partition table
- `n` - Create new partition
- `d` - Delete partition
- `t` - Change partition type
- `w` - Write changes and exit
- `q` - Quit without saving
- `m` - Show help menu

```bash
# View partition types
sudo fdisk /dev/sdb
# Press 'l' to list all partition types
```

### Creating Partitions with cfdisk (Interactive GUI)

```bash
# Open cfdisk interactive partition tool
sudo cfdisk /dev/sdb
```

cfdisk features:
- Select **GPT** for modern systems (supports >2TB disks)
- Select **DOS (MBR)** for legacy compatibility
- Navigate with arrow keys
- Use **New** to create partitions
- Use **Delete** to remove partitions
- Use **Resize** to change partition size
- Use **Type** to change partition type
- Use **Write** to commit changes (type "yes" to confirm)
- Use **Quit** to exit

```bash
# Example: Creating partitions in cfdisk
# 1. Select "Free space"
# 2. Press [New]
# 3. Enter size (e.g., "8G" for 8 gigabytes)
# 4. Press [Type] to change partition type (e.g., "Linux swap" for swap)
# 5. Press [Write] and type "yes" to confirm
# 6. Press [Quit] to exit
```

### Creating Partitions with parted

```bash
# Create GPT partition table
sudo parted /dev/sdb mklabel gpt
```

```bash
# Create partition
sudo parted /dev/sdb mkpart primary ext4 0% 50%
```

```bash
# Delete partition
sudo parted /dev/sdb rm 1
```

### Partition Type Changes

```bash
# Change partition type in fdisk
sudo fdisk /dev/sdb
# Use 't' command, then enter type code
# Common types:
# 82 - Linux swap
# 83 - Linux filesystem
# 8e - Linux LVM
# ef - EFI System
```

---

## Swap Space Management

### Viewing Swap Information

```bash
# Show active swap spaces with details
swapon --show
```

```bash
# Display memory and swap usage
free -h
```

```bash
# View swap usage in /proc
cat /proc/swaps
```

### Creating Swap Partition

```bash
# Format partition as swap
sudo mkswap /dev/vdb3
```

```bash
# Enable swap partition
sudo swapon /dev/vdb3
```

```bash
# Verify swap is active
swapon --show
```

```bash
# Disable swap partition
sudo swapoff /dev/vdb3
```

### Creating Swap File

```bash
# Create 128 MB swap file using dd
sudo dd if=/dev/zero of=/swap bs=1M count=128
```

```bash
# Create 2 GB swap file with progress indicator
sudo dd if=/dev/zero of=/swap bs=1M count=2048 status=progress
```

```bash
# Set correct permissions (security)
sudo chmod 600 /swap
```

```bash
# Format file as swap space
sudo mkswap /swap
```

```bash
# Enable swap file
sudo swapon --verbose /swap
```

```bash
# Verify all active swap areas
swapon --show
```

### Making Swap Persistent

```bash
# Edit fstab for permanent swap configuration
sudo nano /etc/fstab
```

Add entries for swap:
```
# For swap partition:
/dev/vdb3    none    swap    sw    0    0

# For swap file:
/swap        none    swap    sw    0    0
```

```bash
# Test fstab entries without rebooting
sudo mount -a
```

### Adjusting Swappiness

```bash
# View current swappiness value (0-100)
cat /proc/sys/vm/swappiness
```

```bash
# Set swappiness temporarily (lower = less swapping)
sudo sysctl vm.swappiness=10
```

```bash
# Set swappiness permanently
sudo nano /etc/sysctl.conf
# Add: vm.swappiness=10
```

```bash
# Apply sysctl changes
sudo sysctl -p
```

---

## Filesystems

### Creating Filesystems

```bash
# Create ext4 filesystem
sudo mkfs.ext4 /dev/sdb1
```

```bash
# Create ext4 with label
sudo mkfs.ext4 -L mydata /dev/sdb1
```

```bash
# Create XFS filesystem
sudo mkfs.xfs /dev/sdb1
```

```bash
# Create Btrfs filesystem
sudo mkfs.btrfs /dev/sdb1
```

```bash
# Create FAT32 filesystem
sudo mkfs.vfat /dev/sdb1
```

```bash
# Create NTFS filesystem
sudo mkfs.ntfs /dev/sdb1
```

### Viewing Filesystem Information

```bash
# Show filesystem disk space usage
df -h
```

```bash
# Show filesystem type
df -T
```

```bash
# Show inodes usage
df -i
```

```bash
# Show all filesystems including pseudo filesystems
df -a
```

```bash
# Display filesystem information for ext4
sudo dumpe2fs /dev/sdb1
```

```bash
# Display filesystem UUID and type
sudo blkid /dev/sdb1
```

```bash
# List all block device attributes
sudo blkid
```

### Checking and Repairing Filesystems

```bash
# Check ext4 filesystem (unmount first!)
sudo fsck /dev/sdb1
```

```bash
# Force check even if filesystem appears clean
sudo fsck -f /dev/sdb1
```

```bash
# Automatically repair errors
sudo fsck -y /dev/sdb1
```

```bash
# Check specific filesystem type
sudo fsck.ext4 /dev/sdb1
sudo fsck.xfs /dev/sdb1
```

```bash
# Check and repair XFS filesystem
sudo xfs_repair /dev/sdb1
```

### Tuning Filesystems

```bash
# Set filesystem label
sudo e2label /dev/sdb1 newlabel
```

```bash
# View ext4 parameters
sudo tune2fs -l /dev/sdb1
```

```bash
# Set reserved blocks percentage (default 5%)
sudo tune2fs -m 1 /dev/sdb1
```

```bash
# Set maximum mount count before check
sudo tune2fs -c 30 /dev/sdb1
```

```bash
# Set check interval (180 days)
sudo tune2fs -i 180d /dev/sdb1
```

---

## Mounting Filesystems

### Manual Mounting

```bash
# Mount filesystem temporarily
sudo mount /dev/sdb1 /mnt
```

```bash
# Mount with specific filesystem type
sudo mount -t ext4 /dev/sdb1 /mnt
```

```bash
# Mount with options
sudo mount -o rw,noexec /dev/sdb1 /mnt
```

```bash
# Unmount filesystem
sudo umount /mnt
```

```bash
# Force unmount (if busy)
sudo umount -f /mnt
```

```bash
# Lazy unmount (detach now, cleanup when possible)
sudo umount -l /mnt
```

```bash
# Show what process is using the mount
sudo lsof /mnt
# Or:
sudo fuser -m /mnt
```

### Persistent Mounting with /etc/fstab

```bash
# Edit fstab file
sudo nano /etc/fstab
```

fstab format: `<device> <mount_point> <fs_type> <options> <dump> <pass>`

Example entries:
```
# Mount by device name
/dev/sdb1    /mnt/data    ext4    defaults    0    2

# Mount by UUID (recommended)
UUID=a1b2c3d4-5678-90ab-cdef-1234567890ab    /mnt/data    ext4    defaults    0    2

# Mount by label
LABEL=mydata    /mnt/data    ext4    defaults    0    2
```

```bash
# Test fstab configuration without rebooting
sudo mount -a
```

```bash
# Remount all filesystems
sudo mount -a -o remount
```

```bash
# Remount specific mount point with new options
sudo mount -o remount,ro /mnt/data
```

### Finding UUID and Labels

```bash
# Display all UUIDs
sudo blkid
```

```bash
# Display UUID for specific device
sudo blkid /dev/sdb1
```

```bash
# Alternative method to find UUID
ls -l /dev/disk/by-uuid/
```

```bash
# Find devices by label
ls -l /dev/disk/by-label/
```

---

## Filesystem and Mount Options

### Common Mount Options

```bash
# Read-only mount
sudo mount -o ro /dev/sdb1 /mnt
```

```bash
# Read-write mount (default)
sudo mount -o rw /dev/sdb1 /mnt
```

```bash
# Prevent execution of binaries
sudo mount -o noexec /dev/sdb1 /mnt
```

```bash
# Prevent SUID bit operations
sudo mount -o nosuid /dev/sdb1 /mnt
```

```bash
# Prevent device file interpretation
sudo mount -o nodev /dev/sdb1 /mnt
```

```bash
# Mount with multiple options
sudo mount -o rw,noexec,nosuid,nodev /dev/sdb1 /mnt
```

### fstab Mount Options

Common options for `/etc/fstab`:
```
defaults        # rw, suid, dev, exec, auto, nouser, async
ro              # Read-only
rw              # Read-write
noexec          # Do not execute binaries
nosuid          # Ignore SUID/SGID bits
nodev           # Do not interpret character/block devices
noauto          # Do not mount automatically at boot
user            # Allow ordinary users to mount
nofail          # Do not report errors if device doesn't exist
_netdev         # Device requires network (wait for network)
```

Example `/etc/fstab` with options:
```
UUID=xxx    /mnt/data    ext4    rw,noexec,nosuid,nodev    0    2
UUID=yyy    /mnt/backup  ext4    defaults,nofail           0    2
UUID=zzz    /mnt/shared  ext4    rw,user,noauto            0    0
```

---

## NFS - Remote Filesystems

### Installing NFS Client

```bash
# Install NFS client (Ubuntu/Debian)
sudo apt install nfs-common
```

```bash
# Install NFS client (RHEL/CentOS)
sudo yum install nfs-utils
```

### Viewing NFS Exports

```bash
# Show exports from NFS server
showmount -e nfs-server.example.com
```

```bash
# Show all mount points on NFS server
showmount -a nfs-server.example.com
```

### Mounting NFS Shares

```bash
# Mount NFS share temporarily
sudo mount -t nfs nfs-server:/export/share /mnt/nfs
```

```bash
# Mount NFS share with version 4
sudo mount -t nfs4 nfs-server:/export/share /mnt/nfs
```

```bash
# Mount NFS with options
sudo mount -t nfs -o rw,soft,intr nfs-server:/share /mnt/nfs
```

### Persistent NFS Mounts

Add to `/etc/fstab`:
```
nfs-server:/export/share    /mnt/nfs    nfs    defaults    0    0
nfs-server:/export/share    /mnt/nfs    nfs    rw,soft,intr,_netdev    0    0
```

### NFS Mount Options

Common NFS-specific options:
- `soft` - Return error if server doesn't respond
- `hard` - Keep trying until server responds (default)
- `intr` - Allow interruption of NFS calls
- `timeo=n` - Timeout in tenths of a second
- `retrans=n` - Number of retransmissions
- `rsize=n` - Read buffer size
- `wsize=n` - Write buffer size
- `_netdev` - Wait for network before mounting

### NFS Server Configuration

```bash
# Install NFS server
sudo apt install nfs-kernel-server
```

```bash
# Edit exports file
sudo nano /etc/exports
```

Example `/etc/exports`:
```
/export/share    192.168.1.0/24(rw,sync,no_subtree_check)
/export/public   *(ro,sync,no_subtree_check)
```

```bash
# Export all shares
sudo exportfs -a
```

```bash
# Restart NFS server
sudo systemctl restart nfs-server
```

```bash
# Show current exports
sudo exportfs -v
```

---

## Network Block Devices (NBD)

### Installing NBD

```bash
# Install NBD tools (Ubuntu/Debian)
sudo apt install nbd-client nbd-server
```

### NBD Client Commands

```bash
# Load NBD kernel module
sudo modprobe nbd
```

```bash
# Connect to NBD server
sudo nbd-client nbd-server.example.com /dev/nbd0
```

```bash
# Connect to NBD server with specific port
sudo nbd-client nbd-server.example.com 10809 /dev/nbd0
```

```bash
# Connect to named export
sudo nbd-client -N exportname nbd-server.example.com /dev/nbd0
```

```bash
# List available exports
sudo nbd-client -l nbd-server.example.com
```

```bash
# Disconnect NBD device
sudo nbd-client -d /dev/nbd0
```

### Using NBD Devices

```bash
# Create filesystem on NBD device
sudo mkfs.ext4 /dev/nbd0
```

```bash
# Mount NBD device
sudo mount /dev/nbd0 /mnt/nbd
```

```bash
# Unmount and disconnect
sudo umount /mnt/nbd
sudo nbd-client -d /dev/nbd0
```

### NBD Server Configuration

```bash
# Edit NBD server configuration
sudo nano /etc/nbd-server/config
```

Example configuration:
```
[generic]
[export1]
    exportname = /export/disk1.img
    readonly = false
    multifile = false
```

```bash
# Start NBD server
sudo systemctl start nbd-server
```

---

## LVM Storage Management

### Physical Volume (PV) Management

```bash
# Create physical volume
sudo pvcreate /dev/sdb1
```

```bash
# Create multiple physical volumes
sudo pvcreate /dev/sdb1 /dev/sdc1
```

```bash
# Display physical volume information
sudo pvdisplay
```

```bash
# List physical volumes (brief)
sudo pvs
```

```bash
# Scan for physical volumes
sudo pvscan
```

```bash
# Remove physical volume
sudo pvremove /dev/sdb1
```

### Volume Group (VG) Management

```bash
# Create volume group
sudo vgcreate vg_data /dev/sdb1
```

```bash
# Create VG with multiple PVs
sudo vgcreate vg_data /dev/sdb1 /dev/sdc1
```

```bash
# Display volume group information
sudo vgdisplay
```

```bash
# List volume groups (brief)
sudo vgs
```

```bash
# Scan for volume groups
sudo vgscan
```

```bash
# Extend volume group (add PV)
sudo vgextend vg_data /dev/sdd1
```

```bash
# Reduce volume group (remove PV)
sudo vgreduce vg_data /dev/sdd1
```

```bash
# Remove volume group
sudo vgremove vg_data
```

### Logical Volume (LV) Management

```bash
# Create logical volume (10GB)
sudo lvcreate -L 10G -n lv_home vg_data
```

```bash
# Create LV using percentage of VG
sudo lvcreate -l 50%VG -n lv_home vg_data
```

```bash
# Create LV using all free space
sudo lvcreate -l 100%FREE -n lv_home vg_data
```

```bash
# Display logical volume information
sudo lvdisplay
```

```bash
# List logical volumes (brief)
sudo lvs
```

```bash
# Scan for logical volumes
sudo lvscan
```

```bash
# Extend logical volume by 5GB
sudo lvextend -L +5G /dev/vg_data/lv_home
```

```bash
# Extend LV to specific size
sudo lvextend -L 20G /dev/vg_data/lv_home
```

```bash
# Extend LV and resize filesystem together
sudo lvextend -L +5G -r /dev/vg_data/lv_home
```

```bash
# Reduce logical volume (DANGER: can cause data loss!)
# 1. Unmount filesystem
sudo umount /home
# 2. Check filesystem
sudo e2fsck -f /dev/vg_data/lv_home
# 3. Resize filesystem first
sudo resize2fs /dev/vg_data/lv_home 15G
# 4. Reduce LV
sudo lvreduce -L 15G /dev/vg_data/lv_home
```

```bash
# Remove logical volume
sudo lvremove /dev/vg_data/lv_home
```

### Resizing Filesystems on LVM

```bash
# Resize ext4 filesystem after extending LV
sudo resize2fs /dev/vg_data/lv_home
```

```bash
# Resize XFS filesystem after extending LV (must be mounted)
sudo xfs_growfs /home
```

### LVM Snapshots

```bash
# Create snapshot (10% of original size for changes)
sudo lvcreate -L 1G -s -n lv_home_snapshot /dev/vg_data/lv_home
```

```bash
# Mount snapshot
sudo mount /dev/vg_data/lv_home_snapshot /mnt/snapshot
```

```bash
# Remove snapshot
sudo lvremove /dev/vg_data/lv_home_snapshot
```

---

## Storage Performance Monitoring

### Disk Usage Commands

```bash
# Display filesystem disk space usage
df -h
```

```bash
# Display disk space with filesystem type
df -hT
```

```bash
# Display inode usage
df -i
```

```bash
# Display directory size
du -sh /home/user
```

```bash
# Display sizes of subdirectories
du -h --max-depth=1 /var
```

```bash
# Sort directories by size
du -h /var | sort -rh | head -10
```

```bash
# Find large files
find / -type f -size +100M -exec ls -lh {} \;
```

### I/O Performance Monitoring

```bash
# Display I/O statistics (install sysstat package first)
iostat
```

```bash
# Display extended I/O statistics
iostat -x
```

```bash
# Continuous monitoring (update every 2 seconds)
iostat -x 2
```

```bash
# Monitor specific device
iostat -x /dev/sda 2
```

```bash
# Display I/O in MB
iostat -m 2
```

```bash
# Monitor I/O by process (requires root)
sudo iotop
```

```bash
# Display only active processes in iotop
sudo iotop -o
```

### Disk Performance Testing

```bash
# Write test (1GB file)
dd if=/dev/zero of=/tmp/testfile bs=1M count=1024 oflag=direct
```

```bash
# Read test
dd if=/tmp/testfile of=/dev/null bs=1M
```

```bash
# Test with hdparm
sudo hdparm -t /dev/sda
```

```bash
# Test cached reads
sudo hdparm -T /dev/sda
```

### System Resource Monitoring

```bash
# Display virtual memory statistics
vmstat 2
```

```bash
# Display detailed disk statistics
vmstat -d
```

```bash
# Monitor disk activity
watch -n 1 'df -h'
```

```bash
# Check if disk is busy
sudo iotop -b -n 1
```

---

## Advanced Filesystem Permissions

### Standard Permissions

```bash
# View file permissions
ls -l filename
```

```bash
# View directory permissions
ls -ld directoryname
```

```bash
# Change permissions (numeric)
chmod 755 filename    # rwxr-xr-x
chmod 644 filename    # rw-r--r--
chmod 600 filename    # rw-------
```

```bash
# Change permissions (symbolic)
chmod u+rwx filename     # Add rwx for user
chmod g+rw filename      # Add rw for group
chmod o-rwx filename     # Remove all for others
chmod a+r filename       # Add read for all
```

```bash
# Change ownership
sudo chown user:group filename
```

```bash
# Change ownership recursively
sudo chown -R user:group directory/
```

```bash
# Change only user
sudo chown user filename
```

```bash
# Change only group
sudo chgrp group filename
```

### Special Permissions

```bash
# Set SUID (Set User ID) - execute as file owner
chmod u+s filename
# Or:
chmod 4755 filename
```

```bash
# Set SGID (Set Group ID) - execute as group owner / inherit group
chmod g+s directory
# Or:
chmod 2755 directory
```

```bash
# Set Sticky Bit - only owner can delete files in directory
chmod +t directory
# Or:
chmod 1777 directory
```

```bash
# View special permissions
ls -l filename
# Output shows 's' for SUID/SGID, 't' for sticky bit
```

### Access Control Lists (ACLs)

```bash
# View ACL for file or directory
getfacl filename
```

```bash
# Set ACL for user
setfacl -m u:username:rwx filename
```

```bash
# Set ACL for group
setfacl -m g:groupname:rx filename
```

```bash
# Remove specific ACL entry
setfacl -x u:username filename
```

```bash
# Remove all ACL entries
setfacl -b filename
```

```bash
# Set default ACL for directory (inherited by new files)
setfacl -d -m u:username:rwx directory/
```

```bash
# Copy ACL from one file to another
getfacl file1 | setfacl --set-file=- file2
```

```bash
# Recursively set ACL
setfacl -R -m u:username:rx directory/
```

### File Attributes

```bash
# View file attributes
lsattr filename
```

```bash
# Make file immutable (cannot be modified/deleted, even by root)
sudo chattr +i filename
```

```bash
# Remove immutable attribute
sudo chattr -i filename
```

```bash
# Make file append-only
sudo chattr +a filename
```

```bash
# Remove append-only attribute
sudo chattr -a filename
```

Common attributes:
- `i` - Immutable (cannot modify/delete)
- `a` - Append only
- `c` - Compressed
- `s` - Secure deletion (overwrite with zeros)
- `u` - Undeletable (can be recovered)

### umask (Default Permissions)

```bash
# View current umask
umask
```

```bash
# View umask in symbolic notation
umask -S
```

```bash
# Set umask (subtracted from 777 for directories, 666 for files)
umask 022    # Results in 755 for dirs, 644 for files
umask 077    # Results in 700 for dirs, 600 for files
```

```bash
# Set umask permanently (add to ~/.bashrc)
echo "umask 022" >> ~/.bashrc
```
