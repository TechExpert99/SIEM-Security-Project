# Incident Report — IR-004

## 1. Incident Summary

| Field | Details |
|-------|---------|
| Incident ID | IR-004 |
| Incident Title | Privilege Escalation — SYSTEM + Persistence via Admin Group |
| Date & Time | 2026-06-26 01:22 to 01:33 (IST) |
| Severity | Critical |
| Status | Resolved |
| Analyst | Nayak |
| Attacker IP | 192.168.128.164 (Kali Linux) |
| Victim IP | 192.168.128.162 (Windows 11 — NAYAK_RO) |
| Starting Privilege | NAYAK_RO\nayak |
| Final Privilege | NT AUTHORITY\SYSTEM |

---

## 2. Description

Using an active Meterpreter session from Phase 3, the attacker escalated
from NAYAK_RO\nayak to NT AUTHORITY\SYSTEM using Named Pipe Impersonation
(getsystem Technique 1). Mimikatz (Kiwi) was loaded confirming SYSTEM
execution. Local exploit suggester revealed 15 persistence vectors. The
pre-existing backdoor "hacker" account was escalated to Administrators group
establishing persistent privileged access. Wazuh detected the Administrators
Group Change as Level 12 alert (Rule 60154).

---

## 3. Attack Commands Used

```bash
# Meterpreter — Privilege Escalation
getsystem
# Result: got system via technique 1 (Named Pipe Impersonation In Memory/Admin)

getuid
# Result: NT AUTHORITY\SYSTEM

load kiwi
creds_all
# Confirmed: [+] Running as SYSTEM

sysinfo
# Computer: NAYAK_RO | OS: Windows 11 24H2+ | Build 26200

run post/multi/recon/local_exploit_suggester
# Result: 15 persistence vectors found

shell

# Windows CMD as SYSTEM
whoami /priv       # SeDebugPrivilege, SeImpersonatePrivilege Enabled
whoami /groups     # Mandatory Label\System Mandatory Level S-1-16-16384
reg query HKLM\SAM # SAM registry accessible

net user hacker    # hacker account created 6/13/2026 — PRE-EXISTING BACKDOOR
net localgroup administrators
net localgroup administrators hacker /add
# Result: The command completed successfully
```

---

## 4. Timeline of Events

| Time (IST) | Event |
|------------|-------|
| 2026-06-26 01:22 | Shell opened — whoami confirms nayak_ro\nayak |
| 2026-06-26 01:22 | whoami /priv — SeImpersonatePrivilege Enabled |
| 2026-06-26 01:33 | load kiwi — Mimikatz loaded successfully |
| 2026-06-26 01:33 | creds_all — Running as SYSTEM confirmed |
| 2026-06-26 01:33 | getsystem — Already running as SYSTEM |
| 2026-06-26 01:33 | getuid — NT AUTHORITY\SYSTEM confirmed |
| 2026-06-26 01:33 | local_exploit_suggester — 15 persistence vectors |
| 2026-06-26 01:33 | whoami /groups — System Mandatory Level confirmed |
| 2026-06-26 01:33 | reg query HKLM\SAM — SAM accessed |
| 2026-06-26 01:33 | net user hacker — Backdoor account discovered (created 6/13) |
| 2026-06-26 01:33 | net localgroup administrators hacker /add — SUCCESS |
| 2026-06-26 11:02 | Wazuh Rule 60154 — Administrators Group Changed (Level 12) |
| 2026-06-26 11:14 | Wazuh 659 total hits recorded |

---

## 5. Privilege Details

### whoami /priv (Key Enabled Privileges)
| Privilege | Description | State |
|-----------|-------------|-------|
| SeDebugPrivilege | Debug programs | **Enabled** |
| SeImpersonatePrivilege | Impersonate a client after authentication | **Enabled** |
| SeCreateGlobalPrivilege | Create global objects | **Enabled** |
| SeChangeNotifyPrivilege | Bypass traverse checking | **Enabled** |

### whoami /groups (Key Groups)
| Group | SID | Note |
|-------|-----|------|
| BUILTIN\Administrators | S-1-5-32-544 | Group owner |
| Mandatory Label\System Mandatory Level | S-1-16-16384 | **SYSTEM confirmed** |

---

## 6. Persistence Vectors (15 Found)

| # | Module | Vulnerability |
|---|--------|--------------|
| 1 | ms16_075_reflection | Vulnerable |
| 2 | accessibility_features_debugger | Likely exploitable |
| 3 | assistive_technology | Likely exploitable |
| 4 | bits | Vulnerable |
| 5 | powershell_profile | Execution policy undefined |
| 6 | registry | Registry writable |
| 7 | registry_active_setup | Registry writable |
| 8 | registry_userinit | Registry likely exploitable |
| 9 | service | Likely exploitable |
| 10 | service_for_user/lock_unlock | Likely exploitable |
| 11 | service_for_user/logon | Likely exploitable |
| 12 | service_for_user/schedule | Likely exploitable |
| 13 | startup_folder | Write access to Startup folder |
| 14 | task_scheduler | Likely exploitable |
| 15 | userinit_mpr_logon_script | Registry path writable |

---

## 7. 🚨 Critical Finding — Pre-existing Backdoor Account

| Field | Value |
|-------|-------|
| Account Name | **hacker** |
| Created | **2026-06-13 1:49:53 PM** (12 days before exercise) |
| Last Logon | 2026-06-13 1:55:58 PM |
| Group BEFORE exercise | Users only |
| Group AFTER attacker | **Administrators** ← added during this exercise |
| Password Expires | 2026-07-25 |
| Account Active | Yes |

This account was NOT created during this exercise. Pre-existing on the system.
Requires immediate investigation and removal.

---

## 8. Indicators of Compromise (IOCs)

| IOC Type | Value |
|----------|-------|
| Escalation Technique | Named Pipe Impersonation (getsystem T1) |
| Tool | Metasploit getsystem + Kiwi/Mimikatz |
| Backdoor Account | hacker (created 2026-06-13) |
| Registry Access | HKLM\SAM |
| MITRE T1134.001 | Token Impersonation/Theft |
| MITRE T1098 | Account Manipulation |
| MITRE T1547 | Boot/Logon Autostart Execution |
| MITRE T1003 | OS Credential Dumping attempt |
| MITRE T1087 | Account Discovery |

---

## 9. Detection Evidence

### Splunk Results Summary

| Query | Result |
|-------|--------|
| Q1 — Sysmon EventCode overview | 2,277 total events (EC1=1886, EC13=185, EC3=135) |
| Q2 — Process creation EC=1 | 1,886 events including malware.exe |
| Q4 — Failed logins EC=4625 | 10 failures from 192.168.128.164 |
| Q5 — Successful logins EC=4624 | 11 successes from 192.168.128.164 |
| Q8 — Attack timeline | Brute Force Success Login labeled |

### Wazuh Dashboard Results

| Metric | Value |
|--------|-------|
| Total alerts | **659 hits** |
| Level 12+ alerts | **110** |
| Authentication failures | **10** |
| Authentication successes | **17** |

### Key Wazuh Rules Fired

| Rule | Description | Level |
|------|-------------|-------|
| **60154** | **Administrators Group Changed** | **12** 🔴 |
| 92213 | Executable file dropped in malware folder | 15 🔴 |
| 92307 | New service in registry (cmd.exe) | 3 |
| 61138 | New Windows Service Created | 5 |
| 92039 | net.exe account discovery | 3 |
| 92031 | Discovery activity executed | 3 |
| 92052 | Windows cmd by abnormal process | 4 |
| 92032 | Suspicious Windows cmd shell | 3 |
| 92021 | PowerShell used to delete files | 3 |

---

## 10. Root Cause Analysis

1. **SeImpersonatePrivilege enabled** — Key prerequisite for Named Pipe attack
2. **No EDR behavioral detection** — getsystem not blocked
3. **Pre-existing backdoor account** — "hacker" active for 12+ days undetected
4. **No admin group change alerting** — Rule 60154 fired but no response
5. **15 persistence vectors** — System not hardened per CIS benchmarks
6. **Credential Guard not enabled** — Mimikatz able to attempt credential access

---

## 11. Response Actions Taken

- [x] SYSTEM access confirmed via getuid
- [x] All privileges documented via whoami /priv
- [x] 15 persistence vectors identified and documented
- [x] Pre-existing hacker backdoor account discovered and documented
- [x] Administrators Group Change detected in Wazuh (Rule 60154)
- [x] Full attack chain from initial access to SYSTEM documented
- [ ] Remove hacker account: `net user hacker /delete`
- [ ] Re-enable Windows Defender
- [ ] Implement account lockout policy
- [ ] Enable Credential Guard
- [ ] Apply CIS Windows 11 Benchmark

---

## 12. Recommendations

1. **Remove hacker account** immediately and audit its origin
2. **Enable Credential Guard** — Blocks Mimikatz/Kiwi attacks
3. **Disable SeImpersonatePrivilege** for non-service accounts
4. **Deploy EDR** — Detect and block getsystem techniques
5. **Account auditing** — Alert on new local accounts and group changes
6. **CIS Benchmarks** — Apply Windows 11 hardening (Wazuh already detecting gaps)
7. **Account lockout** — 5 attempts max, 30 minute lockout
8. **Block outbound port 4444** — Prevent reverse shell callbacks
