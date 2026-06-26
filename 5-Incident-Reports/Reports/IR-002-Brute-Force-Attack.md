# Incident Report — IR-002

## 1. Incident Summary

| Field | Details |
|-------|---------|
| Incident ID | IR-002 |
| Incident Title | SMB Brute Force Attack — Credential Compromise |
| Date & Time | 2026-06-25 13:07 to 13:11 (IST) |
| Severity | High |
| Status | Resolved |
| Analyst | Nayak |
| Attacker IP | 192.168.128.164 (Kali Linux) |
| Victim IP | 192.168.128.162 (Windows 11 — NAYAK_RO) |
| Target Account | administrator |
| Tool Used | CrackMapExec v1.x |

---

## 2. Description

An attacker from 192.168.128.164 (Kali Linux) performed an SMB brute force
attack against the Windows 11 victim machine (192.168.128.162) targeting the
local administrator account over port 445. The attack used a custom wordlist
of 12 passwords. Multiple failed authentication attempts were followed by a
successful login using the correct password, giving the attacker full
administrative (Pwn3d!) access to the target machine.

Wazuh automatically correlated the failed logins into a "Multiple Windows
Logon Failures" alert (Rule 60204, Level 10) and also flagged the successful
login as a possible Pass-the-Hash attack via NTLM authentication.

---

## 3. Attack Commands Used

```bash
# Step 1 — Verify credentials manually
crackmapexec smb 192.168.128.162 -u administrator -p 2020
# Result: [+] Nayak_RO\administrator:2020 (Pwn3d!)

# Step 2 — Full brute force with wordlist
crackmapexec smb 192.168.128.162 -u administrator -p /tmp/passwords.txt 2>&1 | tee brute-force-results.txt
```

**Wordlist used:**
```
admin, password, 123456, Password1, administrator,
letmein, welcome, nayak, nayak123, 2020, Password@123, qwerty
```

---

## 4. Timeline of Events

| Time (IST) | Event |
|------------|-------|
| 2026-06-25 13:06:19 | User account changed on victim (admin account activated) |
| 2026-06-25 13:06:30 | net.exe account discovery command initiated |
| 2026-06-25 13:06:30 | Discovery activity via PowerShell detected |
| 2026-06-25 13:07:01 | First successful remote logon — Administrator via NTLM |
| 2026-06-25 13:10:04 | Brute force wordlist attack begins |
| 2026-06-25 13:11:01 | Multiple failed logon attempts (EventCode 4625 x 9) |
| 2026-06-25 13:11:15 | Wazuh Rule 60204 fires — Multiple Logon Failures (Level 10) |
| 2026-06-25 13:11:17 | Successful logon — administrator:2020 (Pwn3d!) |
| 2026-06-25 13:11:19 | Wazuh Rule 92652 — Pass-the-Hash alert triggered |
| 2026-06-25 13:11:19 | Special privileges assigned to new logon (Rule 67028) |

---

## 5. Attack Results (Attacker View)

```
SMB  192.168.128.162  445  NAYAK_RO  [*] Windows 11 / Server 2025 Build 26100
SMB  192.168.128.162  445  NAYAK_RO  [-] administrator:admin - TIMEOUT
SMB  192.168.128.162  445  NAYAK_RO  [-] administrator:password - TIMEOUT
SMB  192.168.128.162  445  NAYAK_RO  [-] administrator:administrator - LOGON_FAILURE
SMB  192.168.128.162  445  NAYAK_RO  [-] administrator:letmein - TIMEOUT
SMB  192.168.128.162  445  NAYAK_RO  [+] administrator:2020 (Pwn3d!) ✅
```

---

## 6. Indicators of Compromise (IOCs)

| IOC Type | Value |
|----------|-------|
| Attacker IP | 192.168.128.164 |
| Target IP | 192.168.128.162 |
| Target Account | administrator |
| Compromised Password | 2020 (weak password) |
| Attack Port | 445 (SMB) |
| Auth Protocol | NTLM V2 |
| Logon Type | 3 (Network) |
| MITRE TTP | T1110.001 — Brute Force: Password Guessing |
| MITRE TTP | T1550.002 — Pass the Hash |
| MITRE TTP | T1078.002 — Valid Accounts: Domain Accounts |

---

## 7. Detection Evidence

### Splunk — EventCode 4625 (Failed Logons)
- **Query:** `index=main source="WinEventLog:Security" EventCode=4625`
- **Events:** 10 failed logon attempts
- **Source IP:** 192.168.128.164
- **Account:** administrator / Nayak_RO
- **Failure Reason:** Unknown user name or bad password
- **Logon Type:** 3 (Network/SMB)

### Splunk — EventCode 4624 (Successful Logon)
- **Query:** `index=main source="WinEventLog:Security" EventCode=4624`
- **Events:** 25 successful logon events
- **Attacker logon confirmed from 192.168.128.164**

### Wazuh Alert Chain

| Rule ID | Description | Level | MITRE |
|---------|-------------|-------|-------|
| 60122 | Logon Failure - Unknown user or bad password | 5 | - |
| 60204 | Multiple Windows Logon Failures | 10 | T1110 |
| 92652 | Successful Remote Logon - possible pass-the-hash | 6 | T1550.002 |
| 67028 | Special privileges assigned to new logon | 3 | T1078 |
| 92657 | NTLM auth - Possible RDP connection | 6 | - |
| 92039 | net.exe account discovery initiated | 3 | T1087 |
| 92033 | Discovery via PowerShell execution | 3 | T1059 |

---

## 8. Root Cause Analysis

1. **Weak password** — Administrator account used password "2020" which was
   easily guessed in a small wordlist of 12 attempts
2. **SMB port 445 exposed** — No firewall rule blocking SMB access from
   internal attacker subnet
3. **Administrator account enabled** — Built-in admin was activated and
   accessible over network
4. **No account lockout policy** — Windows allowed unlimited login attempts
   without locking the account

---

## 9. Response Actions Taken

- [x] Identified attacker IP (192.168.128.164) in Splunk 4625 events
- [x] Confirmed successful compromise via EventCode 4624
- [x] Wazuh auto-detected Multiple Logon Failures (Level 10)
- [x] MITRE ATT&CK TTPs mapped by Wazuh automatically
- [x] Full attack chain documented
- [ ] Reset administrator password to complex value
- [ ] Implement account lockout policy (5 attempts / 30 min lockout)
- [ ] Block SMB from untrusted IPs via firewall

---

## 10. Recommendations

1. **Strong password policy** — Minimum 12 characters, complexity required
2. **Account lockout** — Lock after 5 failed attempts for 30 minutes
3. **Disable built-in administrator** — Use named admin accounts instead
4. **Block SMB externally** — Restrict port 445 to trusted IPs only
5. **Enable MFA** — For all privileged accounts

---

## 11. Splunk SPL Queries Used

```spl
-- Failed logon events
index=main source="WinEventLog:Security" EventCode=4625

-- Successful logon events  
index=main source="WinEventLog:Security" EventCode=4624

-- Combined brute force view
index=main source="WinEventLog:Security" (EventCode=4625 OR EventCode=4624)
| eval Status=if(EventCode=4625,"FAILED","SUCCESS")
| table _time, ComputerName, Account_Name, Source_Network_Address, Status
| sort _time
```
