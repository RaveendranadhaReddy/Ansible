# RHEL 9 → RHEL 10 Upgrade Inhibitors - Resolution Guide

## 🔴 Critical Inhibitor Found

### **Problem: CPU Microarchitecture Incompatibility**

**Inhibitor Message:**
```
Current x86-64 microarchitecture is unsupported in RHEL10
RHEL10 requires CPU compatible with x86-64-v3 instruction set or higher
Missing flags: avx2, bmi1, bmi2, fma, abm, movbe
Detected CPU: IVYBRIDGE
```

---

## **Issue Analysis**

### What This Means:
- RHEL 10 requires **x86-64-v3** CPU instruction set
- Your system has an **Intel IvyBridge** processor
- IvyBridge lacks required CPU flags (AVX2, BMI1, BMI2, FMA, ABM, MOVBE)
- **This is a hardware limitation, not software**

### CPU Support Matrix:
```
✅ RHEL 8/9:  x86-64-v1 (basic, very old CPUs)
✅ RHEL 9:    x86-64-v2 (Nehalem, Westmere, Sandy Bridge, Ivy Bridge)
❌ RHEL 10:   x86-64-v3 (Haswell or newer)

Your CPU: Intel Ivy Bridge (released 2012)
Required: Intel Haswell or newer (released 2013)
```

---

## **Solution Options**

### **Option 1: Upgrade to RHEL 9 (Recommended for Production)**

Instead of upgrading to RHEL 10, upgrade from RHEL 8 → RHEL 9.

**Modify playbook for RHEL 9:**
```yaml
- name: Run leapp upgrade to RHEL 9
  command: leapp upgrade --target 9
  async: 5400
  poll: 60
```

**Why RHEL 9:**
- ✅ Supports your Ivy Bridge CPU
- ✅ Supported until May 2032
- ✅ Modern, stable, production-ready
- ✅ Same playbook logic works

---

### **Option 2: Skip CPU Check (NOT Recommended)**

**⚠️ WARNING: Use only for testing/dev environments**

```bash
# On target system before upgrade
leapp upgrade --no-rhsm --el9 --skip-kernel-workarounds
```

**Why This is Risky:**
- ❌ System may not boot after upgrade
- ❌ CPU might not support RHEL 10 instructions
- ❌ Can lead to mysterious system failures
- ❌ No support from Red Hat

---

### **Option 3: Migrate to Newer Hardware**

If you must use RHEL 10:
- Upgrade to Intel Haswell or newer CPU
- Or AMD EPYC / Ryzen (all support x86-64-v3)
- Plan VM migration if virtualized

---

## **Recommended Action: RHEL 8 → RHEL 9**

### Step 1: Check Current Version
```bash
cat /etc/redhat-release
uname -r
```

### Step 2: Updated Playbook for RHEL 9

```yaml
---
- name: RHEL 9 Upgrade - Corrected Playbook
  hosts: all
  become: yes
  serial: 1
  gather_facts: yes
  vars:
    precheck_min_disk_root_mb: 5120
    precheck_min_memory_mb: 1024
    upgrade_reboot_after: true
    upgrade_reboot_timeout: 900
    target_rhel_version: "9"

  tasks:
    #################################################
    # PHASE 1: PRECHECK
    #################################################
    - name: Verify current OS version is RHEL 8
      assert:
        that:
          - ansible_distribution == "RedHat"
          - ansible_distribution_major_version == "8"
        fail_msg: "This playbook requires RHEL 8. Current version: {{ ansible_distribution_major_version }}"

    - name: Check root filesystem disk space
      shell: df -m / | tail -1 | awk '{print $4}'
      register: root_space
      changed_when: false

    - name: Validate sufficient disk space
      assert:
        that:
          - root_space.stdout | int >= precheck_min_disk_root_mb
        fail_msg: "Insufficient disk space"

    - name: Check available memory
      assert:
        that:
          - ansible_memtotal_mb >= precheck_min_memory_mb

    - name: Verify Red Hat subscription
      command: subscription-manager identity
      changed_when: false

    #################################################
    # PHASE 2: INSTALL LEAPP
    #################################################
    - name: Install leapp upgrade utility
      package:
        name: leapp-upgrade
        state: present
      retries: 3
      delay: 10

    - name: Run leapp preupgrade report
      command: leapp preupgrade
      register: leapp_preupgrade
      failed_when: leapp_preupgrade.rc not in [0, 1]
      changed_when: false

    - name: Display preupgrade results
      debug:
        msg: "{{ leapp_preupgrade.stdout }}"

    - name: Check for inhibitors
      shell: grep -i "inhibitor" /var/log/leapp/leapp-report.txt || echo "No inhibitors"
      register: inhibitors
      changed_when: false
      failed_when: false

    - name: Alert on critical inhibitors
      fail:
        msg: "Leapp detected inhibitors. Review /var/log/leapp/leapp-report.txt"
      when: "'Inhibitor' in inhibitors.stdout"

    #################################################
    # PHASE 3: EXECUTE LEAPP UPGRADE TO RHEL 9
    #################################################
    - name: Execute leapp upgrade to RHEL 9
      block:
        - name: Run leapp upgrade
          command: leapp upgrade --target 9
          register: leapp_upgrade_result
          async: 5400
          poll: 60
          changed_when: leapp_upgrade_result.rc == 0

        - name: Display upgrade output
          debug:
            msg: "{{ leapp_upgrade_result.stdout }}"

      rescue:
        - name: Capture error logs
          command: cat /var/log/leapp/leapp-report.txt
          register: error_logs
          ignore_errors: true

        - name: Display errors
          debug:
            msg: "{{ error_logs.stdout }}"

        - name: Fail with error details
          fail:
            msg: "Leapp upgrade to RHEL 9 failed"

    #################################################
    # PHASE 4: WAIT FOR REBOOT
    #################################################
    - name: Wait for system to reboot
      wait_for_connection:
        delay: 30
        timeout: 900

    - name: Re-gather facts after upgrade
      setup:

    #################################################
    # PHASE 5: VALIDATION
    #################################################
    - name: Verify RHEL 9 upgrade
      assert:
        that:
          - ansible_distribution_major_version == "9"
        fail_msg: "Still on RHEL {{ ansible_distribution_major_version }}"

    - name: Check kernel
      command: uname -r
      register: running_kernel
      changed_when: false

    - name: Display upgrade summary
      debug:
        msg: |
          ✓ Upgrade Complete!
          OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Kernel: {{ running_kernel.stdout }}
```

### Step 3: Run Updated Playbook

```bash
ansible-playbook osupdate.yaml -v
```

---

## **Other Non-Critical Issues Found**

### 1. ✅ Berkeley DB (libdb) - MEDIUM
```
Issue: libdb deprecated in RHEL 9, removed in RHEL 10
Impact: Applications using libdb won't work
Action: Not critical for RHEL 9 upgrade
Fix: Migrate if upgrading to RHEL 10 later
```

### 2. ✅ SELinux Mode - LOW
```
Issue: SELinux will switch to permissive mode
Impact: None, required for upgrade
Action: Re-enable SELinux after upgrade:
  semanage permissive -d unconfined_t
  setenforce 1
```

### 3. ✅ Repository Exclusions - INFO
```
Issue: Some CodeReady repos excluded for RHEL 10
Impact: None for RHEL 9
Action: No action needed
```

---

## **Command Reference**

### Check Inhibitors Before Upgrade:
```bash
cat /var/log/leapp/leapp-report.txt
cat /var/log/leapp/leapp-report.json
```

### View Detailed CPU Info:
```bash
# Check CPU model
cat /proc/cpuinfo | grep "model name"

# Check CPU flags
cat /proc/cpuinfo | grep flags
```

### Force Skip CPU Check (⚠️ Not Recommended):
```bash
leapp upgrade --el9 --skip-kernel-workarounds
```

---

## **Timeline & Support**

| Version | Released | Support Ends |
|---------|----------|--------------|
| RHEL 8  | May 2019 | May 2024 ❌ |
| RHEL 9  | May 2022 | May 2032 ✅ |
| RHEL 10 | May 2024 | May 2034 |

**Your Situation:** With Ivy Bridge CPU, you're limited to RHEL 9 max.

---

## **Final Recommendation**

✅ **Upgrade to RHEL 9** (supported until 2032)
- Use the updated playbook above with `--target 9`
- Your CPU fully supports RHEL 9
- No hardware limitations
- Production-ready and stable

❌ **Do NOT attempt RHEL 10** without newer hardware

---

## **Quick Steps**

1. Replace playbook with RHEL 9 version above
2. Run: `ansible-playbook osupdate.yaml -v`
3. System will reboot automatically
4. Verify: `cat /etc/redhat-release`
5. Done! ✅

