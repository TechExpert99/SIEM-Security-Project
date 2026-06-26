# 🌐 Network Topology

## Network: 192.168.128.0/24 (VMware NAT)

```
                        INTERNET
                            │
                    ┌───────▼────────┐
                    │  VMware NAT    │
                    │  Gateway       │
                    │ 192.168.128.2  │
                    └───────┬────────┘
                            │
              NAT Network: 192.168.128.0/24
          ┌─────────────────┼─────────────────┐
          │                 │                 │
  ┌───────▼───────┐ ┌───────▼───────┐ ┌──────▼────────┐
  │  Kali Linux   │ │  Windows 11   │ │    Ubuntu     │
  │  (Attacker)   │ │   (Victim)    │ │    (SIEM)     │
  │               │ │               │ │               │
  │ 192.168.128   │ │ 192.168.128   │ │ 192.168.128   │
  │    .164       │ │    .162       │ │    .163       │
  │               │ │               │ │               │
  │ Tools:        │ │ Tools:        │ │ Tools:        │
  │ • Nmap        │ │ • Sysmon      │ │ • Splunk      │
  │ • Hydra       │ │ • Wazuh Agent │ │ • Wazuh Mgr   │
  │ • Metasploit  │ │ • Splunk UF   │ │               │
  └───────────────┘ └───────────────┘ └───────────────┘
        RED TEAM          TARGET            BLUE TEAM
```

## Attack Flow
```
Kali (Attacker) ──── attacks ────► Windows (Victim)
                                        │
                                   logs forwarded
                                        │
                                        ▼
                               Ubuntu (Splunk + Wazuh)
                               detects & alerts
```

## Port Reference

| Service | Port | Protocol | Direction |
|---------|------|----------|-----------|
| Wazuh Agent → Manager | 1514 | TCP | Win → Ubuntu |
| Splunk UF → Indexer | 9997 | TCP | Win → Ubuntu |
| Splunk Web UI | 8000 | TCP | Browser → Ubuntu |
| Wazuh Web UI | 443 | HTTPS | Browser → Ubuntu |
| SMB (attack target) | 445 | TCP | Kali → Windows |
| RDP (attack target) | 3389 | TCP | Kali → Windows |
