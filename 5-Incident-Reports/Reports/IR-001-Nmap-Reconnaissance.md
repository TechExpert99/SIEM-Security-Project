# Incident Report — IR-001

## 1. Incident Summary

| Field | Details |
|-------|---------|
| Incident ID | IR-001 |
| Incident Title | Network Reconnaissance — Nmap Port Scan |
| Date & Time | 2026-06-24 12:14 to 2026-06-25 02:25 (IST) |
| Severity | Low |
| Status | Resolved |
| Analyst | Nayak |
| Attacker IP | 192.168.128.164 (Kali Linux) |
| Victim IP | 192.168.128.162 (Windows 11 — NAYAK_RO) |

---

## 2. Description

An attacker from 192.168.128.164 (Kali Linux) performed network reconnaissance
against the victim machine 192.168.128.162 (Windows 11). Two Nmap scans were
executed: a host discovery ping sweep across the entire subnet, followed by an
aggressive service/version detection scan targeting all ports on the victim.

The scans revealed open ports 135 (RPC), 139 (NetBIOS), and 445 (SMB), along
with OS fingerprinting identifying the victim as Microsoft Windows 11 24H2
with hostname NAYAK_RO.

---

## 3. Attack Commands Used

```bash
# Phase 1 — Host Discovery (ping sweep)
nmap -sn 192.168.128.0/24

# Phase 2 — Full port + service scan
nmap -sV -sC -O -p- 192.168.128.162

# Phase 3 — Aggressive scan (saved to file)
nmap -A 192.168.128.162 -oN nmap-aggressive-scan.txt
```

---

## 4. Timeline of Events

| Time (IST) | Event |
|------------|-------|
| 2026-06-24 12:14 | Nmap ping sweep launched from Kali (192.168.128.164) |
| 2026-06-24 12:14 | 6 hosts discovered on 192.168.128.0/24 subnet |
| 2026-06-24 12:17 | Full port scan (-p-) started against 192.168.128.162 |
| 2026-06-24 12:35 | Full scan completed — 103 seconds, 3 open ports found |
| 2026-06-25 02:25 | Aggressive scan (-A) launched, completed in 31 seconds |
| 2026-06-25 12:08 | Detection confirmed in Splunk (57 EventCode=3 events) |
| 2026-06-25 12:09 | Wazuh alerts reviewed — 360 total alerts on agent |

---

## 5. Nmap Scan Results (Attacker View)

| Port | State | Service | Version |
|------|-------|---------|---------|
| 135/tcp | open | msrpc | Microsoft Windows RPC |
| 139/tcp | open | netbios-ssn | Microsoft Windows NetBIOS |
| 445/tcp | open | microsoft-ds | SMB (signing enabled) |

**OS Detected:** Microsoft Windows 11 24H2  
**Hostname:** NAYAK_RO  
**MAC Address:** 00:0C:29:9D:B7:AC (VMware)  
**SMB Security:** Message signing enabled and required  

---

## 6. Indicators of Compromise (IOCs)

| IOC Type | Value |
|----------|-------|
| Attacker IP | 192.168.128.164 |
| Victim IP | 192.168.128.162 |
| Victim Hostname | NAYAK_RO |
| Tool Used | Nmap 7.99 |
| Ports Scanned | All 65535 TCP ports |
| Ports Found Open | 135, 139, 445 |

---

## 7. Detection Evidence

### Splunk Detection
- **Query:** EventCode=3 with rex field extraction
- **Events Found:** 57 Sysmon network connection events
- **Source:** WinEventLog:Microsoft-Windows-Sysmon/Operational
- **Host:** Nayak_RO

### Wazuh Detection
- **Total Alerts:** 360 (last 24 hours)
- **Level 12+ Alerts:** 68
- **Key Rules Triggered:**
  - Rule 92052 — Windows command prompt started by abnormal process (Level 4)
  - Rule 92032 — Suspicious Windows cmd shell execution (Level 3)
  - Rule 92213 — Executable file dropped in malware folder (Level 15) 🔴

---

## 8. Root Cause Analysis

The victim machine had port 445 (SMB) exposed and accessible from the internal
network with no host-based firewall blocking inbound connections from the
attacker's subnet. No alerting was configured for port scanning behavior prior
to this exercise.

Sysmon was logging network connections (EventCode 3) but Splunk field extraction
was not configured, requiring manual rex parsing to surface connection details.

---

## 9. Response Actions Taken

- [x] Identified all open ports on victim via SIEM analysis
- [x] Confirmed attacker IP (192.168.128.164) in logs
- [x] Reviewed Wazuh alerts for correlated detections
- [x] Documented attack timeline and IOCs
- [ ] Block attacker IP at firewall (lab exercise — not applied)
- [ ] Restrict SMB/RPC exposure via Windows Firewall rules

---

## 10. Recommendations

1. **Enable host-based firewall rules** — Block inbound connections on ports 135,
   139, 445 from untrusted subnets
2. **Configure Splunk field extractions** — Add proper transforms for Sysmon XML
   fields so EventCode, SourceIp, DestinationIp parse automatically
3. **Create Splunk alert** — Trigger alert when a single source IP hits more than
   50 unique destination ports within 60 seconds
4. **Monitor SMB exposure** — Port 445 open internally is acceptable but should
   be alerted on if accessed from unexpected IPs

---

## 11. Splunk SPL Queries Used

```spl
-- Network connection events with parsed fields
index=main source="WinEventLog:Microsoft-Windows-Sysmon/Operational"
| rex field=_raw "<EventID>(?<EventCode>\d+)</EventID>"
| rex field=_raw "<Data Name='SourceIp'>(?<SourceIp>[^<]+)</Data>"
| rex field=_raw "<Data Name='DestinationIp'>(?<DestinationIp>[^<]+)</Data>"
| rex field=_raw "<Data Name='DestinationPort'>(?<DestinationPort>[^<]+)</Data>"
| rex field=_raw "<Data Name='Image'>(?<Image>[^<]+)</Data>"
| search EventCode=3
| table _time, host, Image, SourceIp, DestinationIp, DestinationPort
| sort _time
```

---

## 12. Screenshots
See `screenshots/` folder:
- `splunk-eventcode3-results.png`
- `splunk-parsed-fields.png`
- `wazuh-360-alerts.png`
- `wazuh-threat-hunting-events.png`
- `kali-nmap-ping-sweep.png`
- `kali-nmap-full-scan.png`
