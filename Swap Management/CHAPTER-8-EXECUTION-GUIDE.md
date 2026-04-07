# CHAPTER 8 - SWAP MANAGEMENT - EXECUTION GUIDE

## 📍 File Location

```
/mnt/user-data/outputs/CHAPTER-8-EXECUTABLE.yml
```

---

## 🚀 QUICK START (4 STEPS)

### Step 1: Check Syntax

```bash
ansible-playbook -i inventory.ini CHAPTER-8-EXECUTABLE.yml --syntax-check
```

### Step 2: Dry-Run (Preview)

```bash
ansible-playbook -i inventory.ini CHAPTER-8-EXECUTABLE.yml --check -v
```

### Step 3: Execute the Playbook

```bash
ansible-playbook -i inventory.ini CHAPTER-8-EXECUTABLE.yml -v
```

### Step 4: Verify Results

```bash
ssh root@server1 'cat /tmp/chapter8_swap_report.txt'
```

---

## 📋 ALL 45+ TASKS AT A GLANCE

| Task | Name | Action |
|------|------|--------|
| 8.1 | Display header | Introduction |
| 8.2 | Swap concepts | Explain swap |
| 8.3 | Check memory | Get mem info |
| 8.4 | Display memory | Show results |
| 8.5 | Check swap | Get swap info |
| 8.6 | Display swap | Show results |
| 8.7 | Swap partition | Explain partition |
| 8.8 | Swap file | Explain file |
| 8.9 | Swap on LVM | Explain LVM |
| 8.10 | Memory tuning | Explain tuning |
| 8.11 | Swap monitoring | Monitoring tips |
| 8.12 | Troubleshooting | Fix issues |
| 8.13 | Hibernation | Explain hibernate |
| 8.14 | Create report | Generate report |
| 8.15 | Report location | Show path |
| 8.16 | Performance metrics | Explain metrics |
| 8.17 | zRAM | Compressed swap |
| 8.45 | Summary | Execution done |

---

## 💾 SWAP TYPES COMPARISON

| Type | Speed | Flexibility | Use Case |
|------|-------|-------------|----------|
| Partition | ⭐⭐⭐ | ⭐ | Production |
| File | ⭐⭐ | ⭐⭐⭐ | Testing |
| LVM | ⭐⭐⭐ | ⭐⭐⭐ | Enterprise |
| zRAM | ⭐⭐⭐⭐ | ⭐ | Mobile/Container |

---

## 🔧 SWAP COMMANDS

### Create Swap Partition

```bash
# Identify partition
lsblk

# Create partition
fdisk /dev/sdb
# n = new, t = type swap, w = write

# Format as swap
mkswap /dev/sdb1

# Enable swap
swapon /dev/sdb1

# Verify
swapon -s
free -h
```

### Create Swap File

```bash
# Create file
fallocate -l 4G /swapfile
# or
dd if=/dev/zero of=/swapfile bs=1G count=4

# Set permissions
chmod 600 /swapfile

# Format as swap
mkswap /swapfile

# Enable swap
swapon /swapfile

# Verify
swapon -s
ls -lh /swapfile
```

### Create Swap on LVM

```bash
# Create logical volume
lvcreate -L 4G -n lv_swap data_vg

# Format as swap
mkswap /dev/data_vg/lv_swap

# Enable swap
swapon /dev/data_vg/lv_swap

# Verify
swapon -s
lvs -o lv_name,lv_size
```

### Add to /etc/fstab

```bash
# Partition
/dev/sdb1         none  swap  sw  0  0

# File
/swapfile         none  swap  sw  0  0

# LVM
/dev/data_vg/lv_swap  none  swap  sw  0  0

# Apply
mount -a
swapon -a
```

### Manage Swap

```bash
# List active swap
swapon -s

# View detailed
free -h

# Disable swap
swapoff /dev/sdb1
swapoff /swapfile

# Monitor continuous
watch -n 1 'free -h'
```

---

## 📊 MEMORY TUNING

### vm.swappiness

```bash
# View current
sysctl vm.swappiness
cat /proc/sys/vm/swappiness

# Set temporary
sysctl -w vm.swappiness=30

# Set persistent (/etc/sysctl.conf)
vm.swappiness = 30

# Apply changes
sysctl -p
```

### Recommended Values

| System | Value | Notes |
|--------|-------|-------|
| Desktop | 60-80 | Acceptable lag |
| Server | 10-30 | Avoid swap |
| Database | 0-10 | Critical latency |
| Development | 50 | Balanced |

---

## 📈 MONITORING SWAP

### Quick View

```bash
free -h              # Simple memory info
free -h -s 5         # Update every 5 seconds
swapon -s            # Swap details
```

### Detailed Monitoring

```bash
# Memory statistics
vmstat 1 10          # 10 samples, 1 second apart

# Top processes
top -o %MEM

# All processes sorted
ps aux --sort=-%mem | head -10

# Per-process swap
for pid in $(pgrep -f); do
    echo -n "$pid: "
    cat /proc/$pid/status | grep VmSwap
done
```

### Understanding vmstat

```
si so = Swap in/out (pages/sec)
- Normal: <10 pages/sec
- Alert: >100 pages/sec
- High values = memory pressure
```

---

## 🆘 TROUBLESHOOTING

### Swap Not Working

```bash
# Check status
swapon -s

# Check fstab
cat /etc/fstab

# Try enable manually
swapon /dev/sdb1

# Check formatting
file /dev/sdb1
```

### System Thrashing (High Swap Activity)

```bash
# Identify cause
vmstat 1 5
top (press M to sort by memory)

# Check memory-hungry processes
ps aux --sort=-%mem | head -5

# Kill offending process
kill -9 PID

# Clear cache
sync; echo 3 > /proc/sys/vm/drop_caches
```

### Can't Disable Swap

```bash
# Check what's using swap
for pid in $(pgrep -f); do
    swap=$(cat /proc/$pid/status | grep VmSwap | awk '{print $2}')
    if [ $swap -gt 0 ]; then echo "$pid: $swap KB"; fi
done

# Kill using processes
kill -9 PID

# Force disable all
swapoff -a
```

---

## 📊 SWAP SIZING GUIDE

### By System Type

| Type | RAM | Swap | Notes |
|------|-----|------|-------|
| Desktop | 16GB | 16-32GB | Hibernation support |
| Server | 64GB | 4-8GB | Fallback only |
| Database | 256GB | 0-10GB | Avoid if possible |
| Dev | 8GB | 16GB | Testing, safety |

### Calculation Examples

```
16GB RAM desktop
→ 16GB swap (hibernate) or 8GB (no hibernate)

64GB server
→ 4GB swap (safety margin)

Development
→ 2x RAM swap (for testing)
```

---

## ⏱️ EXECUTION TIME

| Section | Time |
|---------|------|
| Tasks 8.1-8.6 (Concepts) | ~2 minutes |
| Tasks 8.7-8.12 (Management) | ~3 minutes |
| Tasks 8.13-8.20 (Advanced) | ~2 minutes |
| Tasks 8.21-8.45 (Report) | ~2 minutes |
| **TOTAL** | **~9-15 minutes** |

---

## 📈 EXPECTED OUTPUT

### Task 8.4 - Memory Status
```
              total        used        free      shared  buff/cache
Mem:           62Gi       4.5Gi       52Gi       1.2Gi       5.2Gi
Swap:           8.0Gi     124Mi       7.9Gi
```

### Task 8.6 - Swap Status
```
Filename                Type        Size    Used  Priority
/dev/sdb1               partition   4G      128M  -1
/swapfile               file        2G      0     -2
```

### Task 8.45 - Summary
```
✓ CHAPTER 8 EXECUTION COMPLETE

MEMORY INFORMATION:
Total RAM: 65536MB
Available: 53248MB

SWAP STATUS:
Enabled: 8192MB total
Used: 128MB free

Status: ✓ SUCCESS
```

---

## ✅ VERIFICATION CHECKLIST

After execution:
- [ ] All 45+ tasks executed successfully
- [ ] Report generated at /tmp/chapter8_swap_report.txt
- [ ] Memory status checked
- [ ] Swap status verified
- [ ] Concepts understood
- [ ] Tuning parameters documented
- [ ] Monitoring procedures ready
- [ ] Troubleshooting tips noted

---

## 🎯 NEXT STEPS

1. ✅ Run this playbook successfully
2. ✅ Review current memory/swap status
3. ✅ Assess swap requirements
4. ✅ Create swap (if needed)
5. ✅ Add to /etc/fstab for persistence
6. ✅ Tune vm.swappiness
7. ✅ Set up monitoring
8. ✅ Configure alerts
9. ✅ Document setup
10. ✅ Test recovery procedures

---

**Ready to Execute Chapter 8!** 🚀

```bash
ansible-playbook -i inventory.ini CHAPTER-8-EXECUTABLE.yml -v
```

