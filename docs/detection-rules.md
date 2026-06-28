# Custom Detection Rules

[← Back to README](../README.md)

---

Custom Wazuh rules are stored in `/var/ossec/etc/rules/local_rules.xml` on the Wazuh manager.

---

## Rule 100001 — SSH Authentication Failure from Specific IP

```xml
<group name="local,syslog,sshd,">
  <rule id="100001" level="5">
    <if_sid>5716</if_sid>
    <srcip>1.1.1.1</srcip>
    <description>sshd: authentication failed from IP 1.1.1.1.</description>
    <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5,</group>
  </rule>
</group>
```

**Logic:** Extends the built-in SSH failure rule (5716) and filters for a specific source IP. Useful for tracking a known bad actor.

---

## Rule 100002 — Brute Force Detection

```xml
<group name="local,windows,authentication_failed,">
  <rule id="100002" level="12" frequency="3" timeframe="300">
    <if_matched_group>authentication_failed</if_matched_group>
    <description>Brute force attack detected — more than 3 logon failures in 5 minutes</description>
    <options>no_full_log</options>
    <mitre>
      <id>T1110</id>
    </mitre>
    <group>brute_force,pci_dss_10.2.4,pci_dss_10.2.5,</group>
  </rule>
</group>
```

**Logic:** Uses Wazuh's `frequency` and `timeframe` attributes to aggregate events. When 3 or more `authentication_failed` events occur within 300 seconds, a level 12 (High) alert is triggered. MITRE T1110 and PCI-DSS tags are applied automatically.

---

## Rule 100003 — Mimikatz Detection

```xml
<group name="local,windows,malware,">
  <rule id="100003" level="15">
    <if_sid>62110</if_sid>
    <match>Mimikatz</match>
    <description>CRITICAL: Mimikatz credential dumping tool detected on endpoint</description>
    <mitre>
      <id>T1003</id>
    </mitre>
    <group>credential_dumping,pci_dss_10.6.1,</group>
  </rule>
</group>
```

**Logic:** Fires when Wazuh rule 62110 (Windows Defender threat detection) triggers AND the threat name contains "Mimikatz". Escalates to level 15 — the maximum severity in Wazuh. Tagged with MITRE T1003 (OS Credential Dumping).

---

## Wazuh Severity Levels

| Level | Severity | Description |
|-------|----------|-------------|
| 1–3 | Informational | Normal system activity |
| 4–7 | Low | Unusual but not immediately threatening |
| 8–11 | Medium | Suspicious activity requiring investigation |
| 12–14 | High | Attack pattern detected |
| 15 | Critical | Confirmed attack or critical threat |

---

## How Rules Are Loaded

After editing `local_rules.xml`, rules are reloaded without restarting the manager:

```bash
/var/ossec/bin/wazuh-control restart
```

Rules can be tested against a sample log using:

```bash
/var/ossec/bin/wazuh-logtest
```
