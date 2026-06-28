# MITRE ATT&CK & Compliance Mapping

[← Back to README](../README.md)

---

## MITRE ATT&CK Coverage

### Techniques Simulated

| Technique ID | Name | Tactic | Detected |
|-------------|------|--------|----------|
| T1595 | Active Scanning (Nmap) | Reconnaissance | Partial |
| T1059 | Command & Scripting Interpreter | Execution | Yes |
| T1003 | OS Credential Dumping | Credential Access | Yes |
| T1110 | Brute Force | Credential Access | Yes |
| T1071 | Application Layer Protocol (C2) | Command & Control | Partial |

### Coverage by Tactic

```
Reconnaissance    ██░░░░░░░░  20%
Initial Access    ░░░░░░░░░░   0%
Execution         ████░░░░░░  40%
Persistence       ░░░░░░░░░░   0%
Privilege Esc.    ░░░░░░░░░░   0%
Defense Evasion   ░░░░░░░░░░   0%
Credential Access ████████░░  80%
Discovery         ██░░░░░░░░  20%
Lateral Movement  ░░░░░░░░░░   0%
C2                ████░░░░░░  40%
Exfiltration      ░░░░░░░░░░   0%
Impact            ░░░░░░░░░░   0%
```

---

## Compliance Mapping

Wazuh automatically tags alerts against compliance frameworks. The detections in this lab map to the following controls:

### PCI-DSS

| Control | Description | Triggered By |
|---------|-------------|-------------|
| 10.2.4 | Log invalid logical access attempts | Rule 100001, 100002 |
| 10.2.5 | Log changes to authentication mechanisms | Rule 100002 |
| 10.6.1 | Review security events daily | Rule 100003 |
| 11.4 | Use intrusion detection techniques | Rule 92213, 92052 |

### HIPAA

| Control | Description | Triggered By |
|---------|-------------|-------------|
| 164.312.b | Audit controls for ePHI access | Rule 100002 |
| 164.312.c | Integrity controls | Rule 100003 |
| 164.306.a | Security standards — risk analysis | Rule 92213 |

### GDPR

| Article | Description | Triggered By |
|---------|-------------|-------------|
| Art. 32 | Security of processing | All critical/high alerts |

---

## Alert-to-Compliance Summary

| Alert | Wazuh Rule | MITRE | PCI-DSS | HIPAA |
|-------|-----------|-------|---------|-------|
| Brute force attack | 100002 | T1110 | 10.2.4, 10.2.5 | 164.312.b |
| Mimikatz detected | 100003 | T1003 | 10.6.1 | 164.312.c |
| Malware on endpoint | 62110 | T1003 | 11.4 | 164.306.a |
| Executable in malware folder | 92213 | T1059 | 11.4 | 164.306.a |
| Abnormal process spawn | 92052 | T1059 | 11.4 | 164.306.a |
