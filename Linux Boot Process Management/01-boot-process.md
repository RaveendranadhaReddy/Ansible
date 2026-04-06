# Chapter 1: Linux Boot Process Management

## Overview
**Author: ROOPA DEVi MIRIYALA** <Br>
**File:** `01-boot-process.yml`  
**Duration:** 5-10 minutes  
**Difficulty:** ⭐ Beginner  
**Prerequisites:** Root/sudo access  
**Platform:** RHEL/CentOS 7,8,9 | Rocky | Alma Linux  

---

## Description

This playbook manages the Linux boot process, including systemd targets configuration, GRUB2 settings, and boot diagnostics. It performs the following operations:

- Checks current boot target (systemd)
- Changes default boot target to multi-user.target
- Manages GRUB2 configuration
- Sets GRUB timeout
- Analyzes boot performance
- Generates comprehensive boot reports

---

## Topics Covered

| Task | Description |
|------|-------------|
| 1.1 | Show current default boot target |
| 1.2 | Display current boot target information |
| 1.3 | Set default boot target to multi-user (CLI mode) |
| 1.4 | Verify target change |
| 1.5 | Display target change result |
| 1.6 | Check GRUB2 configuration file exists |
| 1.7 | Display GRUB file status |
| 1.8 | Backup GRUB configuration before changes |
| 1.9 | Display backup status |
| 1.10 | View current GRUB configuration |
| 1.11 | Display GRUB configuration |
| 1.12 | Set GRUB timeout to 5 seconds |
| 1.13 | Regenerate GRUB2 config file |
| 1.14 | Display GRUB regeneration result |
| 1.15 | Check last boot logs (dmesg - errors/warnings) |
| 1.16 | Display boot errors and warnings |
| 1.17 | Get last 20 boot log entries |
| 1.18 | Display journal boot logs |
| 1.19 | Check system uptime |
| 1.20 | Display system uptime |
| 1.21 | Gather comprehensive boot information |
| 1.22 | Display comprehensive boot summary |
| 1.23 | Create boot configuration report |
| 1.24 | Display report location |
| 1.25 | Show final status |

---

## systemd Targets Explained

### Available Targets

```
multi-user.target      = CLI mode (runlevel 3) - DEFAULT FOR SERVERS
graphical.target       = GUI mode (runlevel 5) - DEFAULT FOR DESKTOPS
rescue.target          = Single-user mode (runlevel 1)
emergency.target       = Emergency shell (runlevel S)
poweroff.target        = System shutdown
reboot.target          = System reboot
halt.target            = System halt
```

### Runlevel Mapping

```
Runlevel 0  ↔ poweroff.target     (Shutdown)
Runlevel 1  ↔ rescue.target       (Single user)
Runlevel 2  ↔ multi-user.target   (Multi-user, no NFS)
Runlevel 3  ↔ multi-user.target   (Multi-user with network)
Runlevel 4  ↔ multi-user.target   (Unused)
Runlevel 5  ↔ graphical.target    (GUI)
Runlevel 6  ↔ reboot.target       (Reboot)
```

---

## GRUB2 Configuration

### Key Files

| File | Purpose |
|------|---------|
| `/etc/default/grub` | Main configuration file (EDIT THIS) |
| `/boot/grub2/grub.cfg` | Generated configuration (DO NOT EDIT) |
| `/etc/default/grub.backup` | Backup created by playbook |

### Important Parameters

```
GRUB_TIMEOUT=5              # Seconds to wait for user input
GRUB_DEFAULT=0              # Default menu entry (0-based index)
GRUB_TIMEOUT_STYLE=menu     # Show timeout (menu/hidden/countdown)
GRUB_CMDLINE_LINUX="..."    # Kernel parameters
```

### Kernel Parameters Examples

```
quiet                       # Suppress boot messages
splash                      # Show splash screen
ro                          # Mount root read-only
rw                          # Mount root read-write
selinux=0                   # Disable SELinux
init=/bin/bash              # Boot to bash shell
```

---

## Playbook Variables

```yaml
vars:
  grub_timeout: 5                    # GRUB menu timeout in seconds
  default_target: multi-user.target  # Default boot target to set
```

### How to Customize

Edit the `vars:` section to change:

```yaml
vars:
  grub_timeout: 10                   # Change timeout to 10 seconds
  default_target: graphical.target   # Change to GUI mode
```

---

## Playbook Sections

### Section 1: Check Current Boot Configuration

**Purpose:** Determine the current boot configuration before making changes

**Tasks:**
- Get current default boot target using `systemctl get-default`
- Display current target information
- Show available target options

**Output:**
```
==========================================
CURRENT BOOT CONFIGURATION
==========================================
Current default target: multi-user.target
Target Description:
  - multi-user.target   = CLI mode (runlevel 3)
  - graphical.target    = GUI mode (runlevel 5)
  - rescue.target       = Single-user mode
  - emergency.target    = Emergency shell
==========================================
```

---

### Section 2: Configure Boot Target

**Purpose:** Set the default boot target to multi-user.target (CLI mode)

**Tasks:**
- Execute `systemctl set-default` command
- Verify the change was applied
- Display before/after comparison

**Output:**
```
==========================================
BOOT TARGET CONFIGURATION
==========================================
Previous target: graphical.target
New target: multi-user.target
Status: CHANGED
==========================================
```

---

### Section 3: GRUB2 Configuration

**Purpose:** Manage GRUB2 bootloader settings

**Tasks:**
1. Check if GRUB2 config file exists
2. Display file status and metadata
3. Backup current GRUB configuration
4. Display current GRUB settings
5. Set GRUB timeout to 5 seconds
6. Regenerate GRUB2 configuration file
7. Display regeneration status

**Key Actions:**
- Backs up `/etc/default/grub` before making changes
- Uses `lineinfile` to modify GRUB_TIMEOUT parameter
- Regenerates config with `grub2-mkconfig` command

**Output:**
```
==========================================
GRUB2 CONFIGURATION STATUS
==========================================
GRUB config file exists: True
File path: /etc/default/grub
File size: 567 bytes
Last modified: 2024-04-06T13:00:00+00:00
==========================================
```

---

### Section 4: Boot Logs and Diagnostics

**Purpose:** Analyze boot process and identify any issues

**Tasks:**
1. Check boot errors and warnings using `dmesg`
2. Get recent journal boot entries (last 20)
3. Check system uptime
4. Display all findings

**Commands Used:**
```bash
dmesg --level=err,warn --time-format=iso    # Boot errors/warnings
journalctl -b -n 20 --no-pager              # Recent boot logs
uptime                                       # System uptime
```

**Output:**
```
==========================================
BOOT ERRORS AND WARNINGS (dmesg)
==========================================
[No errors or warnings found in boot logs]
==========================================

SYSTEM UPTIME INFORMATION
==========================================
11:45:23 up 5 days, 3:21, 1 user, load average: 0.12, 0.15, 0.14
==========================================
```

---

### Section 5: Comprehensive Boot Summary

**Purpose:** Provide complete boot analysis and performance metrics

**Tasks:**
1. List all systemd targets
2. Analyze boot time using `systemd-analyze`
3. Show slowest services to load
4. Display comprehensive summary

**Commands Used:**
```bash
systemctl list-units --type=target --all    # List targets
systemd-analyze time                        # Boot time
systemd-analyze blame                       # Slowest services
```

**Output:**
```
==========================================
COMPREHENSIVE BOOT SUMMARY
==========================================

SYSTEMD TARGETS:
UNIT                                   LOAD   ACTIVE SUB    DESCRIPTION
basic.target                           loaded active active Basic System
multi-user.target                      loaded active active Multi-User System
...

BOOT TIME ANALYSIS:
Startup finished in 2.345s (kernel) + 1.234s (userspace) = 3.579s

SLOWEST SERVICES TO LOAD:
 5.234s systemd-journal-flush.service
 3.123s NetworkManager.service
 2.456s sshd.service
...

UPTIME:
11:45:23 up 5 days, 3:21, 1 user, load average: 0.12, 0.15, 0.14

==========================================
BOOT CONFIGURATION COMPLETE
==========================================
```

---

### Section 6: Post-Execution Tasks

**Purpose:** Create audit trail and summary report

**Tasks:**
1. Create comprehensive boot configuration report
2. Save report to `/tmp/chapter1_boot_report.txt`
3. Display report location
4. Show final execution summary

**Report Content:**
```
==========================================
LINUX BOOT PROCESS CONFIGURATION REPORT
==========================================
Generated on: 2024-04-06T13:45:23Z
Hostname: server1
OS: RedHat 8.5

BOOT CONFIGURATION:
Current Default Target: multi-user.target
GRUB Timeout: 5 seconds
GRUB Configuration File: True

BOOT LOGS:
- Errors/Warnings: 0 entries
- Recent Journal: 20 entries

UPTIME:
11:45:23 up 5 days, 3:21, 1 user, load average: 0.12, 0.15, 0.14

BOOT TIME:
Startup finished in 2.345s (kernel) + 1.234s (userspace) = 3.579s

==========================================
```

---

## How to Run the Playbook

### Step 1: Validate Syntax

```bash
ansible-playbook -i inventory.ini playbooks/01-boot-process.yml --syntax-check
```

**Expected Output:**
```
playbook: playbooks/01-boot-process.yml
```

### Step 2: Dry-Run (Check Mode)

```bash
ansible-playbook -i inventory.ini playbooks/01-boot-process.yml --check
```

**This shows what will change WITHOUT making actual changes**

### Step 3: Execute the Playbook

```bash
ansible-playbook -i inventory.ini playbooks/01-boot-process.yml -v
```

**Use `-v` for normal verbosity, `-vv` for more, `-vvv` for maximum**

### Step 4: Execute on Specific Host

```bash
ansible-playbook -i inventory.ini playbooks/01-boot-process.yml --limit server1 -v
```

### Step 5: Execute with Maximum Detail

```bash
ansible-playbook -i inventory.ini playbooks/01-boot-process.yml -vvvv
```

---

## Manual Verification Commands

After the playbook completes, verify the changes:

### Check Boot Target

```bash
# View current default target
systemctl get-default

# List all available targets
systemctl list-units --type=target --all

# Show which target is active
systemctl list-units --type=target --state=active
```

**Expected Output:**
```
multi-user.target
```

### Check GRUB Configuration

```bash
# View GRUB timeout setting
cat /etc/default/grub | grep GRUB_TIMEOUT

# View entire GRUB configuration
cat /etc/default/grub

# Compare with backup
diff /etc/default/grub /etc/default/grub.backup
```

**Expected Output:**
```
GRUB_TIMEOUT=5
```

### Check Boot Logs

```bash
# View current boot logs
journalctl -b

# View last 50 lines of current boot
journalctl -b -n 50

# View previous boot
journalctl -b -1

# View errors only
journalctl -b -p err

# View boot with timestamps
journalctl -b --no-pager
```

### Check Boot Time Performance

```bash
# Show total boot time
systemd-analyze time

# Show slowest services
systemd-analyze blame

# Show services starting in parallel
systemd-analyze plot > boot-diagram.svg

# Show critical chain (critical path)
systemd-analyze critical-chain
```

**Example Output:**
```
Startup finished in 2.345s (kernel) + 1.234s (userspace) = 3.579s
```

### Check System Uptime

```bash
# Show uptime
uptime

# Show last reboot time
who -b

# Show detailed uptime information
systemctl show-environment | grep UPTIME
```

**Example Output:**
```
11:45:23 up 5 days, 3:21, 1 user, load average: 0.12, 0.15, 0.14
```

### View Generated Report

```bash
# View the report created by playbook
cat /tmp/chapter1_boot_report.txt

# Check report on remote server
ssh root@server1 'cat /tmp/chapter1_boot_report.txt'

# Collect report from all servers
for server in server1 server2 server3; do
  echo "=== $server ==="
  scp root@$server:/tmp/chapter1_boot_report.txt ./chapter1_report_$server.txt
done
```

---

## Troubleshooting

### Issue: Target Change Not Applied

**Problem:** Boot target still shows old value

**Solution:**
```bash
# Check if change actually happened
systemctl get-default

# If not changed, try manually
systemctl set-default multi-user.target

# Verify
systemctl get-default

# Check if symbolic link is correct
ls -l /etc/systemd/system/default.target
```

### Issue: GRUB2 Config Not Modified

**Problem:** GRUB_TIMEOUT is still the old value

**Solution:**
```bash
# Check if file was modified
cat /etc/default/grub | grep GRUB_TIMEOUT

# Manually edit if needed
vim /etc/default/grub

# Regenerate GRUB config
grub2-mkconfig -o /boot/grub2/grub.cfg

# Reboot to apply (optional, changes take effect at next boot)
reboot
```

### Issue: Boot Logs Show Errors

**Problem:** `dmesg` or `journalctl` shows errors

**Solution:**
```bash
# View all boot errors
dmesg | grep -i error

# Search for specific errors
journalctl -b -p err

# Check for hardware issues
dmesg | grep -i "firmware"
dmesg | grep -i "hardware"

# Check for driver issues
dmesg | grep -i "driver"
```

### Issue: Playbook Execution Fails

**Problem:** Ansible reports errors

**Solution:**
```bash
# Run with maximum verbosity
ansible-playbook -i inventory.ini playbooks/01-boot-process.yml -vvvv

# Check ansible logs
tail -f /tmp/ansible.log

# Verify connectivity
ansible all -i inventory.ini -m ping

# Test SSH access
ssh -i ~/.ssh/ansible_key root@server1 'uptime'

# Check sudo access
ansible all -i inventory.ini -m command -a 'sudo uptime'
```

---

## Key Concepts Explained

### systemd vs init

Modern Linux systems use **systemd** instead of traditional init scripts:

```
Traditional: runlevels (0-6)
Modern: targets (.target files)

Same concept, different implementation
```

### Boot Targets

**multi-user.target** (Server Default)
- CLI-only mode
- No GUI
- All network services
- Best for servers

**graphical.target** (Desktop Default)
- Includes multi-user.target
- Adds GUI (GNOME, KDE, etc.)
- Best for workstations

### GRUB2 Boot Process

```
1. BIOS/UEFI initialization
2. Boot loader (GRUB2) loads
3. GRUB displays menu (waits GRUB_TIMEOUT seconds)
4. User selects kernel or timeout occurs
5. Kernel loads and initializes
6. systemd starts and reaches target
```

### Performance Metrics

**Boot Time Components:**
```
- Kernel time: Hardware initialization
- Userspace time: Systemd and services
- Total: Sum of both
```

**Optimization Tips:**
```
- Disable unnecessary services
- Use SSD instead of HDD
- Reduce GRUB_TIMEOUT if not needed
- Check slowest services with systemd-analyze blame
```

---

## Best Practices

### Before Running

✅ Read playbook completely  
✅ Test on non-production first  
✅ Backup current GRUB config  
✅ Have recovery plan ready  
✅ Schedule maintenance window  

### During Execution

✅ Monitor playbook output carefully  
✅ Check for error messages  
✅ Verify each section completes  
✅ Note any warnings  

### After Execution

✅ Review generated report  
✅ Verify all changes applied  
✅ Test boot functionality  
✅ Document changes made  
✅ Archive report for audit  

---

## Reference Commands

### View Current Boot Configuration

```bash
# Current target
systemctl get-default

# GRUB settings
cat /etc/default/grub

# Boot logs
journalctl -b
dmesg

# System info
uptime
uname -a
```

### Modify Boot Configuration

```bash
# Change boot target
systemctl set-default multi-user.target

# Edit GRUB
vim /etc/default/grub

# Regenerate GRUB
grub2-mkconfig -o /boot/grub2/grub.cfg

# Reboot (if needed)
reboot
```

### Monitor Boot Performance

```bash
# Total boot time
systemd-analyze time

# Slowest services
systemd-analyze blame

# Boot diagram
systemd-analyze plot | head -20

# Critical services
systemd-analyze critical-chain
```

---

## Related Files

| File | Purpose |
|------|---------|
| `ansible.cfg` | Ansible configuration |
| `inventory.ini` | Host definitions |
| `README.md` | Project guide |
| `EXECUTION_GUIDE.md` | Execution instructions |
| `/tmp/chapter1_boot_report.txt` | Generated report |

---

## Next Steps

After completing Chapter 1:

1. ✅ Review the generated report
2. ✅ Verify boot changes on all servers
3. ✅ Document any modifications
4. ✅ Test system reboot (if applicable)
5. ✅ Proceed to Chapter 2: User & Group Management

---

## Summary

**Chapter 1** provides comprehensive management of the Linux boot process:

- ✅ Configures systemd boot targets
- ✅ Manages GRUB2 bootloader settings
- ✅ Analyzes boot performance
- ✅ Generates audit reports
- ✅ Provides complete diagnostics

**Total Execution Time:** 5-10 minutes  
**Total Tasks:** 25  
**Difficulty:** ⭐ Beginner  

---


**Chapter 1 Complete! Ready for Chapter 2?** 🚀
