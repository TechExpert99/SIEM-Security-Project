# 🖥️ SOC Home Lab — Setup Guide

## Lab Overview

| Machine | OS | IP Address | Role | Platform |
|--------|-----|------------|------|----------|
| Kali Linux | Kali Rolling | 192.168.128.164 | Attacker / Red Team | VMware |
| Windows 11 | Win 11 Home 24H2 | 192.168.128.162 | Victim / Target | VMware |
| Ubuntu | Ubuntu (SIEM) | 192.168.128.163 | Monitoring & Detection | VMware |

**Hypervisor:** VMware Workstation 17 Pro  
**Network Type:** NAT (192.168.128.0/24)  
**Hostname (Windows):** NAYAK_RO  

---

## Log Flow Architecture

```
┌─────────────────────┐         ┌──────────────────────────────────┐
│   Windows 11 Victim │         │        Ubuntu SIEM               │
│   192.168.128.162   │         │      192.168.128.163             │
│                     │         │                                  │
│  ┌───────────────┐  │  TCP    │  ┌──────────┐  ┌─────────────┐  │
│  │    Sysmon     │  │ ──────► │  │  Wazuh   │  │   Splunk    │  │
│  │  (Event logs) │  │  1514   │  │ Manager  │  │ Enterprise  │  │
│  └───────┬───────┘  │         │  └──────────┘  └─────────────┘  │
│          │          │  TCP    │                                  │
│  ┌───────▼───────┐  │ ──────► │  ┌──────────────────────────┐   │
│  │ Splunk UF +   │  │  9997   │  │  Splunk Indexer (main)   │   │
│  │ Wazuh Agent   │  │         │  └──────────────────────────┘   │
│  └───────────────┘  │         │                                  │
└─────────────────────┘         └──────────────────────────────────┘
          ▲
          │
┌─────────────────────┐
│   Kali Linux        │
│   192.168.128.164   │
│   (Attacker)        │
└─────────────────────┘
```

---

## Tools Installed

### Windows 11 Victim (192.168.128.162)
- **Sysmon v10.2** — Advanced system monitoring
  - Logs: Process Create, Network Connections, DNS Queries, File Create
  - Config: SwiftOnSecurity sysmon-config
- **Wazuh Agent v4.14.5** — Forwards logs to Wazuh Manager
- **Splunk Universal Forwarder** — Forwards logs to Splunk

### Ubuntu SIEM (192.168.128.163)
- **Wazuh Manager** — Threat detection & alerting engine
- **Splunk Enterprise 10.4.0** — SIEM & log search platform
  - Index: `main`
  - Sourcetype: `WinEventLog:Microsoft-Windows-Sysmon/Operational`

### Kali Linux Attacker (192.168.128.164)
- **Nmap 7.99** — Network scanning
- **Hydra** — Brute force attacks
- **Metasploit Framework** — Exploitation
- **msfvenom** — Payload generation

---

## Wazuh Agent Configuration

File location on Windows: `C:\Program Files (x86)\ossec-agent\ossec.conf`

Key settings:
- **Server:** 192.168.128.163:1514 (TCP)
- **Crypto:** AES
- **Sysmon logs:** Microsoft-Windows-Sysmon/Operational
- **Security logs:** Enabled (with filters)
- **FIM (File Integrity Monitoring):** Enabled
- **SCA:** Enabled (runs every 12h)
- **Active Response:** Enabled

---

## Splunk Configuration

- **Web UI:** http://192.168.128.163:8000
- **Index:** main
- **Forwarder port:** 9997
- **Log source:** WinEventLog:Microsoft-Windows-Sysmon/Operational

---

## Verification Checklist

- [x] Sysmon64 service Running on Windows
- [x] Wazuh Agent active — Agent ID 002 (NAYAK_RO)
- [x] Splunk receiving 1,273+ Sysmon events
- [x] All 3 VMs reachable on 192.168.128.0/24
- [x] Event Viewer showing 36,985+ Sysmon events

---

## Sysmon Events Monitored

| Event ID | Description |
|----------|-------------|
| 1 | Process Creation |
| 3 | Network Connection |
| 11 | File Created |
| 22 | DNS Query |
| 4625 | Failed Logon (Security) |
| 4672 | Special Privileges Assigned |
| 4688 | New Process Created |
