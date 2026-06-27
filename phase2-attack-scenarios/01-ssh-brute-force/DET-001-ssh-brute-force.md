# Detection Spec: SSH Brute Force

**Spec ID:** DET-001  
**MITRE Technique:** T1110.001  
**Status:** ✅ VALIDATED  
**Last Tested:** 2026-06-27  

---

## Telemetry Source

```
Host:        kali (192.168.100.4)
Log source:  /var/log/auth.log
Agent:       Wazuh Agent 004
Protocol:    SSH (port 22)
```

---

## Detection Condition

```
IF: failed SSH login attempts > 5
FROM: same source IP
WITHIN: 60 seconds
THEN: trigger brute force alert
```

---

## Wazuh Rules

```xml
<!-- Rule 5760: Brute force detected -->
<rule id="5760" level="10">
  <if_matched_sid>5700</if_matched_sid>
  <same_source_ip/>
  <description>sshd: brute force trying to get access to the system</description>
  <mitre>
    <id>T1110</id>
  </mitre>
</rule>

<!-- Rule 5758: Max auth attempts exceeded -->
<rule id="5758" level="8">
  <if_sid>5700</if_sid>
  <match>error: maximum authentication attempts exceeded</match>
  <description>sshd: maximum authentication attempts exceeded</description>
</rule>
```

---

## Attack Executed

```bash
# Tool: Hydra
# Command run on Kali Linux
hydra -l root -P /usr/share/wordlists/fasttrack.txt ssh://192.168.100.4 -t 4

# Results
[22][ssh] Attempting 262 passwords
[STATUS] 262 tasks completed
Duration: ~2 minutes
```

---

## Raw Auth Log (Sample)

```
Jun 27 01:15:32 kali sshd[58700]: Failed password for root from 192.168.100.4 port 48210 ssh2
Jun 27 01:15:33 kali sshd[58700]: Failed password for root from 192.168.100.4 port 48210 ssh2
Jun 27 01:15:34 kali sshd[58700]: Failed password for root from 192.168.100.4 port 48210 ssh2
Jun 27 01:15:35 kali sshd[58700]: Failed password for root from 192.168.100.4 port 48210 ssh2
Jun 27 01:15:36 kali sshd[58700]: Failed password for root from 192.168.100.4 port 48210 ssh2
Jun 27 01:15:37 kali sshd[58825]: error: maximum authentication attempts exceeded for root from 192.168.100.4 port 48210 ssh2 [preauth]
Jun 27 01:15:37 kali sshd[58825]: Disconnecting authenticating user root 192.168.100.4 port 48210: Too many authentication failures [preauth]
```

---

## Wazuh Alert Output

```json
{
  "timestamp": "2026-06-27T01:15:37.265+0000",
  "rule": {
    "id": "5760",
    "level": 10,
    "description": "sshd: brute force trying to get access to the system",
    "mitre": {
      "technique": ["Brute Force"],
      "id": ["T1110"],
      "tactic": ["Credential Access"]
    }
  },
  "agent": {
    "id": "004",
    "name": "kali",
    "ip": "192.168.100.4"
  },
  "data": {
    "srcip": "192.168.100.4",
    "dstuser": "root"
  },
  "location": "/var/log/auth.log"
}
```

---

## Detection Outcome

| Metric | Result |
|--------|--------|
| Detected | ✅ YES |
| Alert rule fired | Rule 5760 (Level 10) |
| Detection latency | <5 seconds |
| False positives | None |
| Attempts before alert | 5 failed logins |

---

## Failure Cases / Edge Cases

| Scenario | Result | Notes |
|----------|--------|-------|
| Slow brute force (1 attempt/min) | ❌ NOT detected | Threshold not met in time window |
| Distributed brute force (multiple IPs) | ❌ NOT detected | Per-IP counting only |
| Valid password found | ❌ NOT detected | Successful login not flagged |

---

## Remediation

```bash
# Harden SSH against brute force
# /etc/ssh/sshd_config

MaxAuthTries 3          # Reduce attempts before disconnect
LoginGraceTime 20       # Reduce connection window
PermitRootLogin no      # Disable direct root login
PasswordAuthentication no  # Require key-based auth

# Add fail2ban for automated IP blocking
sudo apt install fail2ban
sudo systemctl enable fail2ban
```

---

## Evidence Files

- `screenshots/01-hydra-attack.png` — Hydra running against SSH
- `screenshots/02-auth-log.png` — Failed login entries in auth.log
- `screenshots/03-wazuh-alert.png` — Rule 5760 firing in Wazuh
