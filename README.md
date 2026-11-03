# Active Directory Home Lab (SIEM-Ready)

A self-contained cybersecurity home lab built from scratch in VirtualBox to practice
Windows Active Directory administration, blue-team monitoring, and offensive security
in a safe, isolated environment.

> **Status:** Core lab complete — four VMs deployed, networked, and a working
> `lab.local` Active Directory domain with a domain-joined Windows 11 client.

---

## 🎯 Goals

- Build a realistic, company-style network from the ground up.
- Learn Active Directory: domain controllers, users, groups, and Group Policy.
- Set up centralized logging / SIEM to detect activity (blue team).
- Have a safe, legal environment to practice security tooling.

---

## 🧱 Lab Architecture

| VM | OS | Role | IP (example) |
|----|----|------|--------------|
| **DC01** | Windows Server 2022 (Desktop Experience) | Domain Controller — Active Directory + DNS | `10.0.2.15` (static) |
| **Workstation** | Windows 11 Pro | Domain-joined client ("employee PC") | `10.0.2.5` |
| **Attacker** | Kali Linux | Security tooling / analysis | DHCP |
| **SIEM** | Ubuntu 26.04 LTS | Log collection / SIEM (planned: Wazuh) | DHCP |

**Network:** All VMs share a single VirtualBox **NAT Network** (`10.0.2.0/24`), isolated
from the host's real home network. The domain controller runs DNS; the client points its
DNS at the DC so it can locate the domain.

```
                      ┌─────────────────────────┐
                      │   VirtualBox NAT Network │
                      │       10.0.2.0/24        │
                      └────────────┬────────────┘
         ┌────────────┬────────────┼────────────┬────────────┐
         │            │            │            │            │
   ┌─────▼─────┐ ┌────▼─────┐ ┌────▼─────┐ ┌────▼─────┐
   │   DC01    │ │Windows 11│ │   Kali   │ │  Ubuntu  │
   │ Server    │ │  Client  │ │ Attacker │ │   SIEM   │
   │ AD + DNS  │ │ (domain) │ │          │ │ (Wazuh)  │
   └───────────┘ └──────────┘ └──────────┘ └──────────┘
```

*(Add a screenshot of your VirtualBox VM list here.)*

---

## 🛠️ Build Steps

### 1. Virtualization setup
- Installed **VirtualBox** + Extension Pack on a Windows 11 host.
- Enabled CPU virtualization (VT-x/AMD-V) in BIOS.
- Created four VMs and allocated RAM/CPU appropriate to each role.

### 2. Operating system installs
- **Windows Server 2022** — Standard Evaluation (Desktop Experience).
- **Windows 11 Pro** — local account created during OOBE (`ms-cxh:localonly`).
- **Kali Linux** — imported the pre-built VirtualBox image.
- **Ubuntu 26.04 LTS** — adjusted display settings (VMSVGA / 128 MB / 3D off) to resolve a graphics driver issue.

### 3. Networking
- Created a VirtualBox **NAT Network** and attached all four VMs to it.
- Verified connectivity and confirmed all hosts shared the `10.0.2.0/24` subnet.

### 4. Active Directory (DC01)
- Renamed the server to `DC01` and assigned a **static IP** (`10.0.2.15`), DNS pointing to itself (`127.0.0.1`).
- Installed the **Active Directory Domain Services** role.
- Promoted the server to a Domain Controller, creating a new forest: **`lab.local`**.
- Created a test domain user (`jsmith`).

### 5. Domain join (Windows 11)
- Pointed the client's DNS at the domain controller (`10.0.2.15`).
- Joined the client to the `lab.local` domain.
- Logged in successfully with a domain account and confirmed the computer object in **Active Directory Users and Computers**.

### 6. Snapshots
- Took `clean-baseline`, `AD-working`, and `domain-joined` snapshots so any state can be restored instantly.

---

## 🐞 Problems Solved (Troubleshooting Log)

Real troubleshooting I worked through — the most valuable part of the project:

| Problem | Root cause | Fix |
|---------|-----------|-----|
| Windows auto-created a `vboxuser` account I couldn't log into | VirtualBox **Unattended Installation** ran automatically | Recreated the VM without attaching the ISO at creation, then attached it afterward |
| Windows Server booted to a text-only screen | Installed the non-"Desktop Experience" (Server Core) edition | Reinstalled and selected **Standard Evaluation (Desktop Experience)** |
| Ubuntu showed `vmwgfx` errors / blank graphical login | Graphics driver vs. hypervisor mismatch | Set graphics to **VMSVGA**, 128 MB video, 3D off; logged in via TTY (`Ctrl+F3`) and installed guest utilities |
| Windows 11 forced a Microsoft account | Modern Win11 OOBE requirement | Used `start ms-cxh:localonly` to create a local account |
| `lab.local` could not be found / "destination host unreachable" | The two Windows VMs were on **plain NAT** (isolated) instead of a shared **NAT Network**, and the client's DNS pointed at the wrong IP | Put all VMs on the same **NAT Network**; pointed the client's DNS at the DC's real IP (`10.0.2.15`) |

---

## 📚 Skills Demonstrated

- Virtualization & VM lifecycle management (VirtualBox, snapshots)
- Windows Server administration & **Active Directory / DNS**
- Windows & Linux installation and configuration
- TCP/IP fundamentals, static IP addressing, DNS, subnetting
- Network isolation and lab design
- Systematic troubleshooting and documentation

---

## 🚀 Roadmap / Next Steps

- [ ] **Group Policy** — enforce password/lockout policies and desktop restrictions.
- [ ] **Wazuh SIEM** on Ubuntu — forward logs from Windows hosts (with Sysmon).
- [ ] **Detection practice** — generate activity and document how it appears in the SIEM.
- [ ] **File Integrity Monitoring** and custom detection rules.
- [ ] Map detections to the **MITRE ATT&CK** framework.

---

## ⚠️ Disclaimer

This lab is fully isolated and used only for self-education. All activity is performed
against systems I own within this lab. Never test against systems you do not own or
lack explicit permission to test.

---

## 📸 Screenshots

*(Add yours here — great candidates:)*
- VirtualBox VM list showing all four machines
- `Active Directory Users and Computers` with the domain and users
- The Windows 11 client logged in with a `LAB\` domain account
- The computer object listed under the domain's Computers container
