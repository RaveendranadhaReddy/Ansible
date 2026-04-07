# CHAPTER 6 - DISK & FILESYSTEM MANAGEMENT - EXECUTION GUIDE

## 📍 File Location

```
/mnt/user-data/outputs/CHAPTER-6-EXECUTABLE.yml
```

---

## 🚀 QUICK START (4 STEPS)

### Step 1: Check Syntax

```bash
ansible-playbook -i inventory.ini CHAPTER-6-EXECUTABLE.yml --syntax-check
```

### Step 2: Dry-Run (Preview Changes)

```bash
ansible-playbook -i inventory.ini CHAPTER-6-EXECUTABLE.yml --check -v
```

### Step 3: Execute the Playbook

```bash
ansible-playbook -i inventory.ini CHAPTER-6-EXECUTABLE.yml -v
```

### Step 4: Verify Results

```bash
ssh root@server1 'cat /tmp/chapter6_storage_report.txt'
```

---

## ⚠️ IMPORTANT WARNINGS

**This chapter covers advanced storage management!**

⚠️ **Test first** on non-production systems  
⚠️ **Backup data** before making changes  
⚠️ **Understand risks** of partitioning changes  
⚠️ **Test restores** before relying on backups  

**Note:** This playbook is mostly informational and demonstrates concepts. Actual partitioning and LVM changes should be done carefully with proper planning.

---

## 📋 ALL 40+ TASKS AT A GLANCE

| Task | Name | Action |
|------|------|--------|
| 6.1 | Display header | Show introduction |
| 6.2 | Disk information | Explain concepts |
| 6.3 | List block devices | Run lsblk |
| 6.4 | Display devices | Show output |
| 6.5 | Get disk usage | Run df -h |
| 6.6 | Display usage | Show results |
| 6.7 | Get inode usage | Run df -i |
| 6.8 | Display inodes | Show results |
| 6.9 | Filesystem types | Explain options |
| 6.10 | Partitioning info | Explain MBR/GPT |
| 6.11 | LVM concepts | Explain LVM |
| 6.12 | Show mounts | List mounts |
| 6.13 | Display mounts | Show details |
| 6.14 | Create directories | Make mount points |
| 6.15 | Display creation | Show results |
| 6.16 | /etc/fstab | Explain config |
| 6.17 | Directory usage | Show du output |
| 6.18 | Display usage | Show details |
| 6.19 | Filesystem checks | Explain fsck |
| 6.20 | Partition strategies | Discuss layout |
| 6.21 | Partition table | Show parted output |
| 6.22 | Display table | Show details |
| 6.23 | Swap space | Explain swap |
| 6.24 | Current swap | Show swapon |
| 6.25 | Display swap | Show details |
| 6.26 | RAID concepts | Explain RAID |
| 6.27 | Disk quotas | Explain quotas |
| 6.28 | Backup strategies | Discuss backup |
| 6.29 | Optimization | Tips & tricks |
| 6.30 | Create report | Generate report |
| 6.31 | Report location | Show path |
| 6.32 | Performance | Monitoring tips |
| 6.33 | Filesystem limits | Explain limits |
| 6.34 | Partition layouts | Show examples |
| 6.35 | LVM management | Commands |
| 6.40 | Execution summary | Final summary |

---

## 💾 STORAGE CONCEPTS

### Partition Table Types

**MBR (Master Boot Record)**
- Legacy, max 4 primary partitions
- Max 2TB disk size
- Compatible with older systems

**GPT (GUID Partition Table)**
- Modern, unlimited partitions
- Max 8EB disk size
- UEFI compatible, recommended

### Filesystem Types

```bash
ext4    - Stable, general purpose (default on RHEL 7)
xfs     - High performance, large files (default on RHEL 8+)
btrfs   - Advanced, snapshots, compression
vfat    - Removable media, USB drives
```

### LVM Components

```
Physical Volume (PV) → Disk or partition
Volume Group (VG)    → Pool of physical volumes
Logical Volume (LV)  → Partition-like device
```

### Mount Point Structure

```
/ - Root (required)
/boot - Boot files
/home - User data
/var - Logs, cache
/tmp - Temporary
/opt - Optional software
swap - Virtual memory
```

---

## 🔧 ESSENTIAL COMMANDS

### Disk Information

```bash
lsblk                          # List block devices
fdisk -l                       # Show partition table
parted -l                      # Parted partition table
df -h                          # Disk space usage
df -i                          # Inode usage
du -sh /path                   # Directory size
mount                          # Show mounts
```

### Filesystem Creation

```bash
mkfs.ext4 /dev/sda1            # Create ext4
mkfs.xfs /dev/sda1             # Create xfs
mkswap /dev/sda1               # Create swap
```

### Mounting

```bash
mount /dev/sda1 /mnt           # Mount filesystem
umount /mnt                    # Unmount
mount -a                       # Mount all from /etc/fstab
```

### Filesystem Checks

```bash
fsck /dev/sda1                 # Check filesystem
fsck -n /dev/sda1              # Dry-run check
tune2fs /dev/sda1              # Tune ext4
```

### LVM Commands

```bash
pvcreate /dev/sdb              # Create PV
vgcreate vg0 /dev/sdb          # Create VG
lvcreate -L 10G -n lv0 vg0    # Create LV
lvextend -L +5G /dev/vg0/lv0  # Extend LV
resize2fs /dev/vg0/lv0        # Resize filesystem
```

### Swap Management

```bash
swapon -s                      # Show swap
free -h                        # Memory info
swapoff /dev/sda1              # Disable swap
swapon /dev/sda1               # Enable swap
```

---

## 📊 PARTITION LAYOUT RECOMMENDATIONS

### Minimal Setup (Desktop)
```
/dev/sda1  /          20GB
/dev/sda2  swap       4GB
/dev/sda3  /home      Rest
```

### Recommended (Server)
```
/dev/sda1  /          30GB
/dev/sda2  /boot      1GB
/dev/sda3  /var       10GB
/dev/sda4  /home      Large
swap       swap       4GB
```

### Advanced (Enterprise)
```
/dev/sda   /          RAID 1
/dev/sda   /boot      RAID 1
/dev/sdb   /var       RAID 5
/dev/sdc   /home      RAID 5
LVM        Additional volumes
```

---

## 🔐 /etc/fstab Configuration

### Format
```
Device  Mount_Point  Filesystem  Options  Dump  Pass
```

### Example
```bash
/dev/sda1         /         ext4     defaults     0     1
/dev/sda2         /boot     ext4     defaults     0     2
/dev/sda3         /home     ext4     defaults,noatime     0     2
/dev/sda4         none      swap     sw           0     0
/dev/cdrom        /media/cd iso9660  noauto,ro    0     0
```

### Mount Options
```
defaults   - Standard options
noatime    - Don't update access time (faster)
ro         - Read-only
rw         - Read-write
noexec     - No execute permission
nosuid     - No setuid bits
```

---

## 📈 MONITORING & MAINTENANCE

### Daily Checks
```bash
df -h                   # Disk usage
df -i                   # Inode usage
mount                   # Mount status
```

### Weekly Tasks
```bash
du -sh /*              # Directory sizes
fsck -n /dev/sda1      # Check filesystems (dry-run)
logrotate -f /etc/logrotate.conf  # Rotate logs
```

### Monthly Tasks
```bash
fsck /dev/sda1         # Full filesystem check
trim_cache=$(du -sh /var/cache/* | sort -rh | head -1)
# Plan for growth
```

---

## ⏱️ EXECUTION TIME

| Section | Time |
|---------|------|
| Tasks 6.1-6.10 (Concepts) | ~2 minutes |
| Tasks 6.11-6.20 (Advanced) | ~3 minutes |
| Tasks 6.21-6.30 (Monitoring) | ~2 minutes |
| Tasks 6.31-6.40 (Report) | ~2 minutes |
| **TOTAL** | **~9-15 minutes** |

---

## 📈 EXPECTED OUTPUT

### Task 6.6 - Disk Usage
```
Filesystem     Size  Used Avail Use% Mounted on
/dev/sda1       50G   20G   30G  40% /
/dev/sda2      200G  100G  100G  50% /home
tmpfs          7.8G     0  7.8G   0% /dev/shm
```

### Task 6.14 - Mount Directory Creation
```
==========================================
MOUNT DIRECTORIES CREATED
==========================================
✓ /mnt/data
  Owner: root:root
  Permissions: 0755

✓ /mnt/backup
  Owner: root:root
  Permissions: 0755

Status: Created
==========================================
```

### Task 6.40 - Final Summary
```
==========================================
✓ CHAPTER 6 EXECUTION COMPLETE
==========================================

CONCEPTS COVERED:
✓ Disk partitioning
✓ Filesystems
✓ LVM
✓ RAID
✓ Backup strategies
✓ Performance monitoring
✓ Filesystem optimization

Status: ✓ SUCCESS
==========================================
```

---

## 🆘 COMMON STORAGE ISSUES

### Disk Full

```bash
# Find what's using space
du -sh /* | sort -rh

# Clean up
rm -rf /var/log/*.old
rm -rf /var/cache/*

# Monitor more carefully
df -h
watch -n 5 'df -h'
```

### Inode Exhaustion

```bash
# Check inode usage
df -i

# Find many small files
find / -type f | wc -l

# Solution: Archive or delete old files
```

### Filesystem Corruption

```bash
# Unmount first
umount /dev/sda1

# Check (read-only)
fsck -n /dev/sda1

# Repair
fsck -y /dev/sda1

# Remount
mount /dev/sda1 /mnt
```

### LVM Issues

```bash
# Check LVM status
pvdisplay
vgdisplay
lvdisplay

# Extend logical volume
lvextend -L +5G /dev/vg0/lv0
resize2fs /dev/vg0/lv0
```

---

## ✅ VERIFICATION CHECKLIST

After execution:
- [ ] All 40+ tasks executed successfully
- [ ] No errors in output
- [ ] Mount directories created
- [ ] Report generated at /tmp/chapter6_storage_report.txt
- [ ] Disk usage monitored
- [ ] Inode usage checked
- [ ] Filesystem status verified
- [ ] Concepts understood

---

## 🎯 NEXT STEPS

1. ✅ Run this playbook successfully
2. ✅ Review disk usage
3. ✅ Plan filesystem layout
4. ✅ Consider LVM if needed
5. ✅ Implement backup strategy
6. ✅ Set up monitoring
7. ✅ Document current setup
8. ✅ Proceed to Chapter 7: Firewall Management

---

**Ready to Execute Chapter 6!** 🚀

```bash
ansible-playbook -i inventory.ini CHAPTER-6-EXECUTABLE.yml -v
```

