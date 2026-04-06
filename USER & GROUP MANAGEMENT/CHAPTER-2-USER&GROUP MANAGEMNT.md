# CHAPTER 2 - USER & GROUP MANAGEMENT - EXECUTION GUIDE

## 📍 File Location

```
/mnt/user-data/outputs/CHAPTER-2-EXECUTABLE.yml
```

---

## 🚀 QUICK START (4 STEPS)

### Step 1: Check Syntax

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml --syntax-check
```

### Step 2: Dry-Run (Preview Changes)

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml --check -v
```

### Step 3: Execute the Playbook

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml -v
```

### Step 4: Verify Results

```bash
ssh root@server1 'cat /tmp/chapter2_user_report.txt'
```

---

## 📋 ALL 27 TASKS AT A GLANCE

| Task | Name | Action |
|------|------|--------|
| 2.1 | Display Chapter 2 Header | Show task header |
| 2.2 | Create system groups | Create 4 groups |
| 2.3 | Display group creation results | Show group info |
| 2.4 | Create system users | Create 4 users |
| 2.5 | Display user creation results | Show user info |
| 2.6 | Verify groups were created | Get group database |
| 2.7 | Display all groups | Show all groups |
| 2.8 | Verify users were created | Get user database |
| 2.9 | Display all users | Show all users |
| 2.10 | Configure password aging | Set pwd policies |
| 2.11 | Display password aging config | Show policies |
| 2.12 | Create .ssh directories | Create SSH dirs |
| 2.13 | Create authorized_keys files | Create SSH auth |
| 2.14 | Display SSH setup status | Show SSH info |
| 2.15 | Create sudoers directory | Create /etc/sudoers.d |
| 2.16 | Configure sudo - wheel group | Set wheel sudo |
| 2.17 | Configure sudo - developers | Set dev sudo |
| 2.18 | Display sudoers config | Show sudo info |
| 2.19 | Verify user creation | Run id command |
| 2.20 | Display user verification | Show id output |
| 2.21 | Verify password aging | Run chage |
| 2.22 | Display password aging verify | Show chage |
| 2.23 | Verify group membership | Run groups |
| 2.24 | Display group membership | Show groups |
| 2.25 | Create management report | Generate report |
| 2.26 | Display report location | Show location |
| 2.27 | Show final summary | Execution summary |

---

## 🎯 GROUPS CREATED

| Group Name | GID | Members | Purpose |
|-----------|-----|---------|---------|
| developers | 1001 | devuser, appuser | Development team |
| sysadmins | 1002 | sysadmin | System admins |
| appgroup | 1003 | appuser | Application user |
| backup | 1004 | backupuser | Backup operations |

---

## 👥 USERS CREATED

### User: devuser
- **UID:** 2001
- **Primary Group:** developers
- **Secondary Groups:** wheel (sudo access)
- **Home:** /home/devuser
- **Shell:** /bin/bash
- **Type:** Regular user (can login)
- **Sudo:** Yes (via wheel group)

### User: sysadmin
- **UID:** 2002
- **Primary Group:** sysadmins
- **Secondary Groups:** wheel (sudo access)
- **Home:** /home/sysadmin
- **Shell:** /bin/bash
- **Type:** Regular user (can login)
- **Sudo:** Yes (via wheel group)

### User: appuser
- **UID:** 2003
- **Primary Group:** appgroup
- **Secondary Groups:** developers
- **Home:** /home/appuser
- **Shell:** /bin/bash
- **Type:** Regular user (can login)
- **Sudo:** Limited (systemctl, docker)

### User: backupuser
- **UID:** 2004
- **Primary Group:** backup
- **Secondary Groups:** None
- **Home:** /home/backupuser (not created)
- **Shell:** /bin/nologin (cannot login)
- **Type:** System user (no interactive login)
- **Sudo:** None

---

## 🔐 PASSWORD POLICIES

```
Minimum days between changes:    0 days
Maximum days until expiration:   90 days
Warning days before expiration:  14 days
```

Applied to: devuser, sysadmin, appuser  
NOT applied to: backupuser (system user)

---

## 🔑 SSH CONFIGURATION

**Directories Created:**
```
/home/devuser/.ssh/              (devuser)
/home/devuser/.ssh/authorized_keys

/home/sysadmin/.ssh/             (sysadmin)
/home/sysadmin/.ssh/authorized_keys

/home/appuser/.ssh/              (appuser)
/home/appuser/.ssh/authorized_keys

(backupuser: no SSH directory, cannot login)
```

**Permissions:**
```
.ssh directory:        0700 (rwx------)
authorized_keys file:  0600 (rw-------)
```

---

## 🛡️ SUDO CONFIGURATION

### wheel group (devuser, sysadmin)
```
%wheel ALL=(ALL) NOPASSWD: ALL
```
**Full sudo access without password prompt**

### developers group (appuser, devuser)
```
%developers ALL=(ALL) NOPASSWD: /usr/bin/systemctl, /bin/systemctl, /usr/bin/docker
```
**Limited sudo access (systemctl and docker only)**

### backup group (backupuser)
```
No sudo access
```

---

## 📊 WHAT GETS CREATED

### Groups (4)
```bash
developers   (GID 1001)
sysadmins    (GID 1002)
appgroup     (GID 1003)
backup       (GID 1004)
```

### Users (4)
```bash
devuser      (UID 2001) - developer with wheel group
sysadmin     (UID 2002) - sysadmin with wheel group
appuser      (UID 2003) - app user with dev group
backupuser   (UID 2004) - backup user (system, no login)
```

### Files Modified/Created
```
/home/devuser/          - Created
/home/devuser/.ssh/     - Created
/home/sysadmin/         - Created
/home/sysadmin/.ssh/    - Created
/home/appuser/          - Created
/home/appuser/.ssh/     - Created
/etc/sudoers.d/wheel-group       - Created
/etc/sudoers.d/developers-group  - Created
/tmp/chapter2_user_report.txt    - Created
```

---

## 🔍 VERIFICATION COMMANDS

### Check if groups exist

```bash
# All groups
getent group

# Specific group
getent group developers

# Count groups
getent group | wc -l
```

### Check if users exist

```bash
# All users
getent passwd

# Specific user
getent passwd devuser

# Count users
getent passwd | wc -l
```

### User details

```bash
# Get user ID and groups
id devuser

# Show user's groups
groups devuser

# Show all groups for user
getent group | grep devuser
```

### Password aging

```bash
# Check password aging for specific user
chage -l devuser

# Expected output:
# Last password change                  : Jan 06, 2024
# Password expires                      : Apr 06, 2024
# Password inactive                     : never
# Account expires                       : never
# Minimum number of days between changes: 0
# Maximum number of days between changes: 90
# Number of days of warning before password expires: 14
```

### Sudo access

```bash
# Check sudo permissions for devuser
sudo -l -U devuser

# Check sudo permissions for appuser
sudo -l -U appuser

# Test running command as devuser
sudo -u devuser whoami
```

### SSH setup

```bash
# Check .ssh directory exists
ls -la /home/devuser/.ssh/

# Check authorized_keys
ls -la /home/devuser/.ssh/authorized_keys

# Expected output:
# drwx------  2 devuser developers 4096 Jan  6 13:45 .ssh
# -rw-------  1 devuser developers    0 Jan  6 13:45 authorized_keys
```

### Sudoers files

```bash
# Check sudoers files
ls -la /etc/sudoers.d/

# Validate sudoers syntax
visudo -c

# Check specific sudoers file
cat /etc/sudoers.d/wheel-group
cat /etc/sudoers.d/developers-group
```

### View report

```bash
# View generated report
cat /tmp/chapter2_user_report.txt

# Copy report
scp root@server1:/tmp/chapter2_user_report.txt ./

# For multiple servers
for server in server1 server2 server3; do
  scp root@$server:/tmp/chapter2_user_report.txt ./chapter2_report_${server}.txt
done
```

---

## 🔄 EXECUTION EXAMPLES

### Example 1: Run on all hosts

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml -v
```

### Example 2: Run on specific host

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml --limit server1 -v
```

### Example 3: Maximum verbosity

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml -vvvv
```

### Example 4: Check mode (dry-run)

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml --check -v
```

### Example 5: Run specific task

```bash
# List all tasks first
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml --list-tasks

# Run specific task
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml --start-at-task "2.2 - Create system groups" -v
```

---

## ⏱️ EXECUTION TIME

| Section | Time |
|---------|------|
| Tasks 2.1-2.3 (Groups) | ~1 minute |
| Tasks 2.4-2.9 (Users) | ~2 minutes |
| Tasks 2.10-2.14 (SSH/Sudo) | ~1 minute |
| Tasks 2.15-2.18 (Verification) | ~1 minute |
| Tasks 2.19-2.27 (Reports) | ~1 minute |
| **TOTAL** | **~5-10 minutes** |

---

## 📈 EXPECTED OUTPUT

### Task 2.3 - Group Creation
```
==========================================
GROUP CREATION RESULTS
==========================================
✓ developers:
  - GID: 1001
  - Status: CREATED
  - Description: Developers Group

✓ sysadmins:
  - GID: 1002
  - Status: CREATED
  - Description: System Administrators Group

... (more groups)
==========================================
```

### Task 2.5 - User Creation
```
==========================================
USER CREATION RESULTS
==========================================
✓ devuser:
  - UID: 2001
  - Primary Group: developers
  - Secondary Groups: wheel
  - Home Directory: /home/devuser
  - Login Shell: /bin/bash
  - Comment: Developer User
  - Status: CREATED

... (more users)
==========================================
```

### Task 2.27 - Final Summary
```
==========================================
✓ CHAPTER 2 EXECUTION COMPLETE
==========================================

EXECUTION STATISTICS:
- Total Tasks Executed: 27
- Host: server1
- OS: RedHat 8.5

GROUPS CREATED: 4
✓ developers (GID: 1001)
✓ sysadmins (GID: 1002)
✓ appgroup (GID: 1003)
✓ backup (GID: 1004)

USERS CREATED: 4
✓ devuser (UID: 2001)
✓ sysadmin (UID: 2002)
✓ appuser (UID: 2003)
✓ backupuser (UID: 2004)

CONFIGURATION:
✓ Password aging: CONFIGURED (90 day max)
✓ SSH directories: CREATED
✓ Sudo access: CONFIGURED
✓ Group membership: ASSIGNED

Status: ✓ SUCCESS
==========================================
```

---

## ⚠️ IMPORTANT NOTES

### Before Running

✅ **Update inventory.ini** with your server IPs  
✅ **Test connectivity** with `ansible all -m ping`  
✅ **Check syntax** with `--syntax-check`  
✅ **Run dry-run** with `--check`  

### What Gets Modified

⚠️ `/etc/passwd` - New users added  
⚠️ `/etc/shadow` - Password hashes  
⚠️ `/etc/group` - New groups added  
⚠️ `/etc/sudoers.d/` - Sudo config added  
⚠️ Home directories - Created with .ssh  

### Safe to Run

✅ Non-destructive (no deletions)  
✅ Idempotent (safe to run multiple times)  
✅ Backed by validation (visudo check)  
✅ No data loss  

### Important Security Notes

⚠️ **Passwords:** Set via password_hash()  
⚠️ **First Login:** Users should change passwords  
⚠️ **SSH Keys:** Deploy keys for key-based auth  
⚠️ **Sudo:** Configured without password prompts  
⚠️ **backupuser:** Cannot login (system account)  

---

## 🆘 TROUBLESHOOTING

### Group Not Created

```bash
# Check if group exists
getent group developers

# Check manually
groupadd -g 1001 developers

# Verify
getent group developers
```

### User Not Created

```bash
# Check if user exists
getent passwd devuser

# Check manually
useradd -u 2001 -g developers -G wheel -m -s /bin/bash devuser

# Verify
id devuser
```

### SSH Directory Not Created

```bash
# Create manually
mkdir -p /home/devuser/.ssh
chown devuser:developers /home/devuser/.ssh
chmod 700 /home/devuser/.ssh

# Create authorized_keys
touch /home/devuser/.ssh/authorized_keys
chown devuser:developers /home/devuser/.ssh/authorized_keys
chmod 600 /home/devuser/.ssh/authorized_keys
```

### Sudo Not Working

```bash
# Validate sudoers syntax
visudo -c

# Check sudo file
cat /etc/sudoers.d/wheel-group

# Test with specific user
sudo -l -U devuser

# Try running command
sudo -u devuser whoami
```

### Password Aging Not Applied

```bash
# Check manually
chage -l devuser

# Apply manually
chage -n 0 -x 90 -w 14 devuser

# Verify
chage -l devuser
```

---

## 🎯 NEXT STEPS

1. ✅ Run this playbook successfully
2. ✅ Verify all users and groups created
3. ✅ Deploy SSH keys to new users
4. ✅ Test sudo access
5. ✅ Have users change passwords on first login
6. ✅ Document user roles and responsibilities
7. ✅ Proceed to Chapter 3: File Permissions & ACL

---

## 📚 RELATED COMMANDS

```bash
# User management
useradd, usermod, userdel
id, groups, getent passwd

# Group management
groupadd, groupmod, groupdel
getent group

# Password management
passwd, chage, pwscore

# Sudo management
visudo, sudo -l, sudoers

# SSH management
ssh-keygen, ssh-copy-id, ssh-add
```

---

## ✅ VERIFICATION CHECKLIST

After execution:
- [ ] All 4 groups created
- [ ] All 4 users created
- [ ] Password aging configured (90 days)
- [ ] SSH directories created (.ssh)
- [ ] authorized_keys files exist
- [ ] Sudo access configured
- [ ] Group membership correct
- [ ] Report generated
- [ ] Manual verification successful

---

**Ready to Execute Chapter 2!** 🚀

```bash
ansible-playbook -i inventory.ini CHAPTER-2-EXECUTABLE.yml -v
```

