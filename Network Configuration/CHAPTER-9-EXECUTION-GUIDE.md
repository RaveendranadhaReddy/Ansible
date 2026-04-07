# CHAPTER 9 - NETWORK CONFIGURATION - EXECUTION GUIDE

## 📍 File Location

```
/mnt/user-data/outputs/CHAPTER-9-EXECUTABLE.yml
```

---

## 🚀 QUICK START (4 STEPS)

### Step 1: Check Syntax
```bash
ansible-playbook -i inventory.ini CHAPTER-9-EXECUTABLE.yml --syntax-check
```

### Step 2: Dry-Run (Preview)
```bash
ansible-playbook -i inventory.ini CHAPTER-9-EXECUTABLE.yml --check -v
```

### Step 3: Execute the Playbook
```bash
ansible-playbook -i inventory.ini CHAPTER-9-EXECUTABLE.yml -v
```

### Step 4: Verify Results
```bash
ssh root@server1 'cat /tmp/chapter9_network_report.txt'
```

---

## 📋 ALL 50+ TASKS

| Task | Name |
|------|------|
| 9.1 | Display header |
| 9.2 | Networking fundamentals |
| 9.3 | Check interfaces |
| 9.4 | Display interfaces |
| 9.5 | Get IP addresses |
| 9.6 | Display IPs |
| 9.7 | Config file locations |
| 9.8 | DHCP vs Static |
| 9.9 | Check DNS |
| 9.10 | Display DNS |
| 9.11 | Routing concepts |
| 9.12 | Check routing |
| 9.13 | Display routes |
| 9.14 | Bonding concepts |
| 9.15 | VLAN concepts |
| 9.16 | Check hostname |
| 9.17 | Display hostname |
| 9.18 | Network security |
| 9.19 | Troubleshooting |
| 9.20 | Create report |
| 9.21 | Report location |
| 9.22 | Packet transmission |
| 9.50 | Summary |

---

## 🔧 ESSENTIAL NETWORKING COMMANDS

### Interface Management
```bash
ip link show              # List interfaces
ip addr show              # Show IP addresses
ip link set eth0 up       # Bring interface up
ip link set eth0 down     # Bring interface down
```

### IP Configuration
```bash
ip addr add 192.168.1.100/24 dev eth0
ip addr del 192.168.1.100/24 dev eth0
ip addr show eth0         # Show eth0 IPs
```

### DNS Configuration
```bash
cat /etc/resolv.conf      # View DNS
nslookup google.com       # DNS lookup
dig google.com            # Detailed DNS
host google.com           # Simple lookup
```

### Routing
```bash
ip route show             # View routes
ip route add 192.168.2.0/24 via 192.168.1.1
ip route del 192.168.2.0/24
route -n                  # Traditional view
```

### Testing Connectivity
```bash
ping 8.8.8.8             # ICMP ping
traceroute google.com    # Path trace
mtr google.com           # Continuous trace
```

---

## 💾 CONFIGURATION FILES

### RHEL/CentOS
```
/etc/sysconfig/network
/etc/sysconfig/network-scripts/ifcfg-eth0
/etc/hostname
/etc/hosts
/etc/resolv.conf
```

### Debian/Ubuntu
```
/etc/network/interfaces
/etc/hostname
/etc/hosts
/etc/resolv.conf
```

---

## 📊 IP ADDRESSING

### CIDR Notation
```
/8  = 256 addresses
/16 = 65,536 addresses
/24 = 256 addresses
/25 = 128 addresses
/30 = 4 addresses
/32 = 1 address
```

### Private IP Ranges
```
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

---

## ✅ VERIFICATION CHECKLIST

After execution:
- [ ] All 50+ tasks executed
- [ ] Report generated at /tmp/chapter9_network_report.txt
- [ ] Network interfaces verified
- [ ] IP configuration checked
- [ ] DNS settings verified
- [ ] Routing table reviewed
- [ ] Hostname configured
- [ ] Connectivity tested

---

**Ready to Execute Chapter 9!** 🚀

```bash
ansible-playbook -i inventory.ini CHAPTER-9-EXECUTABLE.yml -v
```

