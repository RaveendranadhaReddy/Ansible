# CHAPTER 7 - LVM MANAGEMENT - EXECUTION GUIDE

## 📍 File Location

```
/mnt/user-data/outputs/CHAPTER-7-EXECUTABLE.yml
```

---

## 🚀 QUICK START (4 STEPS)

### Step 1: Check Syntax

```bash
ansible-playbook -i inventory.ini CHAPTER-7-EXECUTABLE.yml --syntax-check
```

### Step 2: Dry-Run (Preview)

```bash
ansible-playbook -i inventory.ini CHAPTER-7-EXECUTABLE.yml --check -v
```

### Step 3: Execute the Playbook

```bash
ansible-playbook -i inventory.ini CHAPTER-7-EXECUTABLE.yml -v
```

### Step 4: Verify Results

```bash
ssh root@server1 'cat /tmp/chapter7_lvm_report.txt'
```

---

## 📋 ALL 50+ TASKS AT A GLANCE

| Task | Name | Action |
|------|------|--------|
| 7.1 | Display header | Introduction |
| 7.2 | Display architecture | Explain LVM |
| 7.3 | Check existing PVs | List PVs |
| 7.4 | Display PVs | Show results |
| 7.5 | Display VGs | List VGs |
| 7.6 | Display VGs detail | Show detail |
| 7.7 | Display LVs | List LVs |
| 7.8 | Display LVs detail | Show detail |
| 7.9 | Sizing concepts | Explain sizing |
| 7.10 | PV management | Explain PV |
| 7.11 | VG management | Explain VG |
| 7.12 | LV creation | Explain LV |
| 7.13 | LV extension | Resize LV |
| 7.14 | Snapshots | Explain snapshots |
| 7.15 | Thin provisioning | Virtual volumes |
| 7.16 | Monitoring | Watch usage |
| 7.17 | Configuration | Config files |
| 7.18 | Disaster recovery | Recover LVM |
| 7.19 | Troubleshooting | Fix issues |
| 7.20 | Best practices | Guidelines |
| 7.21 | LVM demo | Setup example |
| 7.22 | Create report | Generate |
| 7.23 | Report location | Show path |
| 7.24 | Advanced features | Striping, mirroring |
| 7.50 | Summary | Execution done |

---

## 🔧 ESSENTIAL LVM COMMANDS

### Physical Volumes

```bash
pvcreate /dev/sdb              # Create PV
pvdisplay                      # Show details
pvs                            # Quick view
pvscan                         # Scan all PVs
pvremove /dev/sdb              # Remove PV
pvmove /dev/sdb /dev/sdc       # Move data
```

### Volume Groups

```bash
vgcreate data_vg /dev/sdb      # Create VG
vgdisplay                      # Show details
vgs                            # Quick view
vgscan                         # Scan all VGs
vgextend data_vg /dev/sdc      # Add PV to VG
vgreduce data_vg /dev/sdc      # Remove PV from VG
vgremove data_vg               # Delete VG
```

### Logical Volumes

```bash
lvcreate -L 10G -n lv_data data_vg     # Create 10GB LV
lvdisplay                              # Show details
lvs                                    # Quick view
lvscan                                 # Scan all LVs
lvextend -L +5G /dev/data_vg/lv_data   # Add 5GB
resize2fs /dev/data_vg/lv_data         # Resize ext4
lvremove /dev/data_vg/lv_data          # Delete LV
```

### Snapshots

```bash
lvcreate -L 1G -s -n snap /dev/data_vg/lv_data
mount /dev/data_vg/snap /mnt/snap
tar -czf backup.tar.gz /mnt/snap/*
lvremove /dev/data_vg/snap
```

### Monitoring

```bash
pvs, vgs, lvs                          # Quick view
pvdisplay, vgdisplay, lvdisplay        # Detailed
lvs -o lv_name,lv_size,data_percent    # Custom output
watch -n 5 'lvs -o lv_name,lv_size'    # Monitor
```

---

## 📊 LVM HIERARCHY

```
Physical Disk (/dev/sdb)
    ↓
Physical Volume (PV)
    ↓
Volume Group (VG) ← Can contain multiple PVs
    ↓
Logical Volume (LV) ← Can have multiple LVs per VG
    ↓
Filesystem (ext4, xfs)
    ↓
Mount Point (/mnt/data)
```

---

## 💾 CREATING LVM STEP BY STEP

### Step 1: Create Physical Volume

```bash
pvcreate /dev/sdb
pvcreate /dev/sdb /dev/sdc /dev/sdd
pvdisplay
```

### Step 2: Create Volume Group

```bash
vgcreate data_vg /dev/sdb
vgcreate data_vg /dev/sdb /dev/sdc
vgdisplay data_vg
```

### Step 3: Create Logical Volume

```bash
lvcreate -L 10G -n lv_storage data_vg
lvcreate -l 50%VG -n lv_half data_vg
lvcreate -l 100%FREE -n lv_all data_vg
lvdisplay
```

### Step 4: Create Filesystem

```bash
mkfs.ext4 /dev/data_vg/lv_storage
mkfs.xfs /dev/data_vg/lv_storage
```

### Step 5: Mount Volume

```bash
mount /dev/data_vg/lv_storage /mnt/storage
df -h /mnt/storage
```

### Step 6: Add to /etc/fstab

```bash
/dev/data_vg/lv_storage  /mnt/storage  ext4  defaults  0  2
mount -a
```

---

## 🔄 EXTENDING LOGICAL VOLUMES

### Step 1: Extend LV

```bash
lvextend -L +5G /dev/data_vg/lv_storage
lvextend -L 20G /dev/data_vg/lv_storage
lvextend -l +100%FREE /dev/data_vg/lv_storage
```

### Step 2: Resize Filesystem

**For ext4:**
```bash
resize2fs /dev/data_vg/lv_storage
resize2fs -p /dev/data_vg/lv_storage (with progress)
```

**For xfs:**
```bash
xfs_growfs /mnt/storage
```

### Step 3: Verify

```bash
lvdisplay /dev/data_vg/lv_storage
df -h /mnt/storage
```

---

## 📸 SNAPSHOTS FOR BACKUP

```bash
# Create snapshot
lvcreate -L 2G -s -n backup /dev/data_vg/lv_data

# Mount snapshot
mount /dev/data_vg/backup /mnt/backup

# Backup files
tar -czf /backup/data.tar.gz /mnt/backup/*

# Cleanup
umount /mnt/backup
lvremove /dev/data_vg/backup
```

---

## 💧 THIN PROVISIONING

### Create Thin Pool

```bash
lvcreate -T -L 50G data_vg/thin_pool
lvcreate -T -L 50G -M 5G data_vg/thin_pool (with metadata)
```

### Create Thin Volumes

```bash
lvcreate -V 100G -T data_vg/thin_pool -n thin_vol1
lvcreate -V 50G -T data_vg/thin_pool -n thin_vol2
lvcreate -V 50G -T data_vg/thin_pool -n thin_vol3
```

### Monitor Pool

```bash
lvs -o lv_name,lv_size,data_percent data_vg
lvdisplay data_vg/thin_pool | grep Data
```

⚠️ **Critical:** Monitor pool usage! If it fills, volumes fail!

---

## 📊 MONITORING & ALERTS

### View Status

```bash
pvs                    # Physical volumes
vgs                    # Volume groups
lvs                    # Logical volumes
```

### Custom Output

```bash
lvs -o lv_name,lv_size,data_percent
vgs -o vg_name,vg_free,vg_size
pvs -o pv_name,pv_used,pv_free
```

### Watch Changes

```bash
watch -n 5 'lvs -o lv_name,lv_size'
watch -n 5 'vgs -o vg_name,vg_free'
```

### Alert Thresholds

- LV > 80% full
- VG < 10% free
- Snapshot > 80% full
- Thin pool > 90% full

---

## 🆘 TROUBLESHOOTING

### VG Not Found

```bash
pvscan                 # Scan for PVs
vgscan                 # Scan for VGs
vgchange -a y          # Activate
```

### Missing PV

```bash
vgreduce --removemissing data_vg
vgchange -a y data_vg
```

### LV Not Accessible

```bash
lvchange -a y /dev/data_vg/lv_data
fsck -n /dev/data_vg/lv_data
mount /dev/data_vg/lv_data /mnt
```

### Snapshot Invalid

```bash
lvs -o lv_name,snap_percent
lvextend -L +1G /dev/data_vg/snap
```

### Thin Pool Exhausted

```bash
lvs data_vg/pool -o data_percent
lvextend -L +10G data_vg/pool
```

---

## 💾 BACKUP & RECOVERY

### Backup Metadata

```bash
vgcfgbackup data_vg
cp /etc/lvm/backup/data_vg /backup/
tar -czf lvm_config.tar.gz /etc/lvm/
```

### Restore Metadata

```bash
vgcfgrestore -f /etc/lvm/backup/data_vg data_vg
vgchange -a y data_vg
```

### Archive History

```bash
ls -la /etc/lvm/archive/
```

---

## ⏱️ EXECUTION TIME

| Section | Time |
|---------|------|
| Tasks 7.1-7.10 (Concepts) | ~2 minutes |
| Tasks 7.11-7.20 (Management) | ~3 minutes |
| Tasks 7.21-7.30 (Advanced) | ~2 minutes |
| Tasks 7.31-7.50 (Report) | ~2 minutes |
| **TOTAL** | **~9-12 minutes** |

---

## ✅ VERIFICATION CHECKLIST

After execution:
- [ ] All 50+ tasks executed successfully
- [ ] No errors in output
- [ ] Report generated at /tmp/chapter7_lvm_report.txt
- [ ] Current LVM status displayed
- [ ] Concepts understood
- [ ] Best practices reviewed
- [ ] Troubleshooting tips noted
- [ ] Recovery procedures documented

---

## 🎯 NEXT STEPS

1. ✅ Run this playbook successfully
2. ✅ Review current LVM configuration
3. ✅ Plan LVM implementation
4. ✅ Create test LVM volumes
5. ✅ Test snapshots
6. ✅ Implement monitoring
7. ✅ Set up backup procedures
8. ✅ Test disaster recovery

---

**Ready to Execute Chapter 7!** 🚀

```bash
ansible-playbook -i inventory.ini CHAPTER-7-EXECUTABLE.yml -v
```

