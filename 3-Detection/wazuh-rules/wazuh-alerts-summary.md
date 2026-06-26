# Wazuh Alerts — Phase 3 Malware Simulation

## Alert Summary (431 total hits)

| Rule ID | Description | Level | MITRE ID | MITRE Tactic |
|---------|-------------|-------|----------|--------------|
| 61638 | Sysmon - Suspicious Process - dllhost.exe | 12 | T1055 | Defense Evasion, Privilege Escalation |
| 92058 | Application Compatibility Database launched | 12 | - | - |
| 92031 | Discovery activity executed | 3 | T1087 | Discovery |
| 92052 | Windows command prompt by abnormal process | 4 | - | - |
| 92657 | Successful Remote Logon - possible pass-the-hash | 6 | T1550.002 | Lateral Movement |
| 67028 | Special privileges assigned to new logon | 3 | T1078 | Privilege Escalation |

## Key Alert Details

### Rule 61638 — Suspicious dllhost.exe (Level 12)
- Process: C:\Windows\System32\dllhost.exe
- MD5: 3ED4BBF7EC264EFC5BDCD3DC0A35ABF1
- Fired 4 times — triggered by Meterpreter process injection

### Rule 92031 — Discovery Activity (Level 3)
- Command: net localgroup administrators
- Parent: net.exe → net1.exe (classic discovery chain)
- User: Nayak_RO\nayak
- MITRE T1087 — Account Discovery

## Raw Alert Files
- wazuh-discovery-activity-alert.json — net localgroup administrators
- wazuh-dllhost-suspicious-process-alert.json — dllhost.exe injection
