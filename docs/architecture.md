# Lab Architecture

[← Back to README](../README.md)

---

## Network Overview

```
┌─────────────────────────────────────────────────────────┐
│                     HOME NETWORK                        │
│                   192.168.1.0/24                        │
│                                                         │
│  ┌──────────────────────┐    ┌───────────────────────┐  │
│  │     Mac Mini         │    │   Windows 11 Laptop   │  │
│  │  Ubuntu 22.04        │    │   (MSI) - Endpoint    │  │
│  │  192.168.1.85        │    │   192.168.1.235       │  │
│  │                      │    │                       │  │
│  │  ┌────────────────┐  │    │  ┌─────────────────┐  │  │
│  │  │ Wazuh Manager  │◄─┼────┼──│  Wazuh Agent    │  │  │
│  │  │ Wazuh Indexer  │  │    │  │  Sysmon         │  │  │
│  │  │ Wazuh Dashboard│  │    │  └─────────────────┘  │  │
│  │  │ (Docker)       │  │    │                       │  │
│  │  └────────────────┘  │    │  ┌─────────────────┐  │  │
│  └──────────────────────┘    │  │ Kali Linux (VM) │  │  │
│                              │  │ 192.168.1.99    │  │  │
│                              │  │ Attacker        │  │  │
│                              │  └─────────────────┘  │  │
│                              └───────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

---

## Components

| Component | Host | IP | Role |
|-----------|------|----|------|
| Wazuh Manager | Mac Mini (Docker) | 192.168.1.85 | Processes logs, applies rules, generates alerts |
| Wazuh Indexer | Mac Mini (Docker) | 192.168.1.85 | Stores events (OpenSearch) |
| Wazuh Dashboard | Mac Mini (Docker) | 192.168.1.85 | Visualization and management |
| Wazuh Agent | Windows 11 Laptop | 192.168.1.235 | Collects and forwards logs |
| Sysmon | Windows 11 Laptop | 192.168.1.235 | Deep endpoint telemetry |
| Kali Linux | VM on Windows Laptop | 192.168.1.99 | Attack simulation |

---

## Communication Flow

```
Wazuh Agent  ──[TCP 1514]──────► Wazuh Manager
Wazuh Agent  ──[TCP 1515]──────► Wazuh Manager (enrollment)
Wazuh Manager──[HTTPS 9200]────► Wazuh Indexer
Browser      ──[HTTPS 443]─────► Wazuh Dashboard
```

---

## Wazuh Stack — Docker Setup

The full Wazuh stack runs on a Mac Mini via Docker Compose:

```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.14.5
cd wazuh-docker/single-node

# Generate SSL certificates
docker-compose -f generate-indexer-certs.yml run --rm generator

# Start the stack
docker-compose up -d
```

**Running containers:**

```
wazuh-manager-1    → ports 1514, 1515, 55000
wazuh-indexer-1    → port 9200
wazuh-dashboard-1  → port 443
```

---

## Wazuh Agent — Windows Installation

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.5-1.msi `
  -OutFile wazuh-agent.msi

msiexec.exe /i wazuh-agent.msi `
  WAZUH_MANAGER='192.168.1.85' `
  WAZUH_AGENT_NAME='windows-laptop' /q

NET START WazuhSvc
```

---

## Sysmon Configuration

Sysmon is installed using the **SwiftOnSecurity** config, providing visibility into:

| Event ID | Description |
|----------|-------------|
| 1 | Process creation (with full command line) |
| 3 | Network connections |
| 7 | Image/DLL loaded |
| 10 | Process access (LSASS injection detection) |
| 11 | File created |
| 12/13 | Registry modifications |

```powershell
.\Sysmon64.exe -accepteula -i sysmonconfig-export.xml
```

---

## Agent Log Collection

The agent is configured to forward Windows Defender and Sysmon events to the manager (`config/agent.conf`):

```xml
<agent_config>
  <localfile>
    <location>Microsoft-Windows-Windows Defender/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>

  <localfile>
    <location>Microsoft-Windows-Sysmon/Operational</location>
    <log_format>eventchannel</log_format>
  </localfile>
</agent_config>
```

---

## Port Reference

| Port | Protocol | Service |
|------|----------|---------|
| 1514 | TCP/UDP | Agent log forwarding |
| 1515 | TCP | Agent enrollment |
| 9200 | HTTPS | OpenSearch API |
| 443 | HTTPS | Wazuh Dashboard |
| 55000 | HTTPS | Wazuh REST API |
