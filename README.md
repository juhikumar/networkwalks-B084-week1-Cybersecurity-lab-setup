# networkwalks-B084-week1-Cybersecurity-lab-setup
Home cybersecurity lab built with VirtualBox and Kali Linux, featuring manual static IP configuration and network setup.
<div align="center">

# 🛡️ Cybersecurity Lab: VirtualBox + Kali Linux

**A private NAT Network lab for ethical hacking and penetration testing practice**

![VirtualBox](https://img.shields.io/badge/VirtualBox-7.x-183A61?style=flat-square&logo=virtualbox&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali%20Linux-2026.2-557C94?style=flat-square&logo=kalilinux&logoColor=white)
![Network](https://img.shields.io/badge/Network-10.0.0.0%2F24-238F89?style=flat-square)
![Level](https://img.shields.io/badge/Level-Beginner-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Phase%201%20Complete-brightgreen?style=flat-square)

</div>

---

## 📌 Overview

This repository documents the first lab of my cybersecurity journey. I built an isolated virtual environment on my own computer, with **Kali Linux** as the attacker machine, connected through a custom **VirtualBox NAT Network** (`10.0.0.0/24`).

Kali uses a **manually assigned static IP** and has full internet access. The network is also designed so that more target machines can be added later for authorized practice.

I'm a second-year B.Sc. Cybersecurity student, and I'm writing this to keep a clear record of how the lab works and what I learned building it.

---

## 🎯 Requirements Completed

- [x] VirtualBox installed as the base hypervisor
- [x] Kali Linux set up as the attacking machine
- [x] Custom NAT Network created with subnet `10.0.0.0/24`
- [x] Clipboard sharing and drag & drop enabled
- [x] Shared folder enabled using the host's `Downloads` folder
- [x] Kali Linux assigned the static IP `10.0.0.2/24`
- [x] Full internet access confirmed from Kali
- [x] Clean snapshot taken after configuration

---

## 🧰 Lab Specifications

| Component | Details |
|---|---|
| Host OS | [Windows 11] |
| Host Hardware | [e.g. 16 GB RAM, Intel core Ultra 5, 1 TB SSD] |
| Hypervisor | VirtualBox [7.2.22] |
| Guest OS | Kali Linux [2026.3] |
| Kali VM RAM | [2048 MB] |
| Network Type | NAT Network (custom) |
| Network Name | `NatNetwork` |
| Subnet | `10.0.0.0/24` |
| Kali IP | `10.0.0.2/24` (manual) |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` |
| Reserved Range for Lab VMs | `10.0.0.2 – 10.0.0.99` |

---

## 🗺️ Network Architecture

```text
 ┌─────────────────────────────────────────────────────┐
 │  Host Machine  [Windows 10/11]                      │
 │                                                     │
 │   ┌─────────────────────────────────────────────┐   │
 │   │ VirtualBox  ─  NatNetwork (10.0.0.0/24)     │   │
 │   │                Gateway: 10.0.0.1            │   │
 │   │                                             │   │
 │   │   ┌──────────────────┐                      │   │
 │   │   │   Kali Linux     │  ← Attacker machine  │   │
 │   │   │   10.0.0.2/24    │                      │   │
 │   │   └──────────────────┘                      │   │
 │   │                                             │   │
 │   │   (Future target VMs: 10.0.0.3 – 10.0.0.99) │   │
 │   └─────────────────────────────────────────────┘   │
 │                         │                           │
 └─────────────────────────┼───────────────────────────┘
                           ▼
                       Internet
```

![Lab diagram](images/lab-diagram.png)

---

## 🪜 Setup Walkthrough

### Step 1: Install 7-Zip
Kali's pre-built VirtualBox image comes as a compressed archive, so I installed 7-Zip first to extract it.

### Step 2: Install VirtualBox
I downloaded VirtualBox from the official site and installed it on my host machine.

### Step 3: Create the NAT Network
In VirtualBox, I opened **File → Tools → Network → NAT Networks** and created a new network:

```text
Name        : NatNetwork
IPv4 Prefix : 10.0.0.0/24
DHCP        : Enabled
IPv6        : Disabled
```

I chose a NAT Network because every VM attached to it can talk to the others while still reaching the internet through NAT. That makes it a good fit for an attacker and target lab.

![Network settings](images/network-settings.png)

### Step 4: Import Kali Linux
I downloaded the official Kali Linux VirtualBox image, extracted it, and imported it into VirtualBox. Then I adjusted the VM settings:

```text
Network Adapter 1 : Attached to NAT Network
Network Name      : NatNetwork
Adapter Type      : Intel PRO/1000 MT Desktop
Promiscuous Mode  : Allow All
RAM               : [2048 MB]
Shared Clipboard  : Bidirectional
Drag and Drop     : Bidirectional
Shared Folder     : Host "Downloads" folder (Auto-mount, Full access)
```

![Kali VM settings](images/kali-vm-settings.png)

### Step 5: Configure the Static IP in Kali
Inside Kali, I opened **Edit Connections**, selected the wired connection, and set IPv4 to **Manual**:

```text
Address : 10.0.0.2
Netmask : 24
Gateway : 10.0.0.1
DNS     : 8.8.8.8
```

![Kali network config](images/kali-network.png)

### Step 6: Take a Snapshot
After confirming everything worked, I took a VirtualBox snapshot named:

```text
Clean Kali - Network Configured
```

This gives me a safe baseline to return to if I break something during later experiments.

---

## ✅ Verification

| Test | Command | Expected Result |
|---|---|---|
| Check IP | `ip a` | Shows `10.0.0.2/24` |
| Ping gateway | `ping -c 4 10.0.0.1` | Replies received |
| Ping internet | `ping -c 4 8.8.8.8` | Replies received |
| DNS lookup | `nslookup google.com` | Domain resolves |
| Tool check | `nmap --version` | Version displayed |
| Shared folder | `ls /media/sf_Downloads` | Host files visible |

![Ping test](images/ping-test.png)

---

## 🐞 Problems I Faced & Fixes

### Problem 1: No internet after setting the static IP
**Cause:** A known issue with newer VirtualBox 7 and Kali 2026.x releases, where NetworkManager's duplicate address detection delays or blocks the connection.

**Fix:**

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

Then I restarted Kali and tested again. 

> 💡 If internet still fails, I double-check that the NAT Network exists, that no other VM uses `10.0.0.2`, and I try `10.0.0.1` as the DNS server.

---

## 📚 Key Takeaways

- **NAT vs NAT Network:** a plain NAT isolates each VM, while a NAT Network lets VMs talk to each other and still reach the internet.
- **Static addressing:** I now understand how the IP address, subnet mask, gateway, and DNS work together.
- **Troubleshooting:** testing in order (IP, then gateway, then internet, then DNS) makes it easy to find where connectivity breaks.
- **Snapshots:** a clean baseline makes it safe to experiment.
- **Documentation:** writing everything down is part of doing security work properly.

---

## 🔭 Phase 2 (Planned)

Additional machines will join the same `10.0.0.0/24` network to build a full attack and defense environment:

| Machine | Planned IP | Status |
|---|---|---|
| Kali Linux (attacker) | 10.0.0.2 | ✅ Done |
| Windows 10 VM | 10.0.0.10 | ⬜ Planned |
| Windows 11 VM | 10.0.0.11 | ⬜ Planned |
| Windows 7 VM | 10.0.0.7 | ⬜ Planned |
| Android VM | 10.0.0.9 | ⬜ Planned |
| Windows Server 2016 | 10.0.0.16 | ⬜ Planned |

Next goals: ping test between all machines, take snapshots of each, and begin Nmap scanning practice.

---
## 📸 Screenshots

![Screenshot 1](Screenshot%202026-10-30%20190741.png)

![Screenshot 2](Screenshot%202026-10-30%20190848.png)

![Screenshot 3](Screenshot%202026-10-30%20191053.png)

![Screenshot 4](Screenshot%202026-10-30%20193500.png)

![Screenshot 5](VirtualBox_kali-linux-2026.3-virtualbox-amd64_FULL_NAME_1.png)

![Screenshot 6](VirtualBox_kali-linux-2026.3-virtualbox-amd64_FULL_NAME_2.png)

![Screenshot 7](VirtualBox_kali-linux-2026.3-virtualbox-amd64_FULL_NAME_3.png)

---

## ⚠️ Ethical Use Disclaimer

This lab is for **education and legal practice only**. I only test systems that I own or have explicit permission to test. Using these tools against anyone else's systems is illegal.

---

## 🔗 Resources

- [7-Zip](https://7-zip.org/download.html)
- [VirtualBox Downloads](https://virtualbox.org/wiki/Downloads)
- [Kali Linux Downloads](https://kali.org/get-kali)

---

## 👤 Author

**[Juhi Kumar]**
B.Sc. Cybersecurity Student (Year 2)


---

<div align="center">

*Part of my cybersecurity learning journey. Updated as the lab grows.*

</div>
