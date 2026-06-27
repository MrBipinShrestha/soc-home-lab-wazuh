# 🛡️ SOC Detection Engineering Lab — Wazuh SIEM

![Wazuh](https://img.shields.io/badge/Wazuh-v4.14.5-blue?style=for-the-badge)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red?style=for-the-badge)
![Focus](https://img.shields.io/badge/Focus-Detection_Engineering-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)

> **Enterprise-grade SOC simulation environment focused on detection engineering, adversary emulation, and SIEM rule validation using Wazuh.**
>
> This lab replicates real-world attacker behavior, validates detection coverage, and documents telemetry gaps with root cause analysis.

---

## 🧠 Engineering Objective

> "Can we reliably detect attacker behavior across identity, endpoint, and persistence layers using SIEM telemetry alone?"

To answer this, I built an end-to-end attack simulation pipeline and measured detection coverage across 5 attack scenarios.

---

## 🏗️ Lab Architecture

```
                ┌──────────────────────────────┐
                │      Wazuh SIEM Manager      │
                │   Ubuntu 22.04 (Core SIEM)   │
                │  Detection + Correlation     │
                └──────────────┬───────────────┘
                               │
                NAT Network (soc-lab 192.168.100.0/24)
                               │
    ┌──────────────────────────┼──────────────────────────┐
    │                          │                          │
┌───▼────────────┐     ┌───────▼───────────┐     ┌───────▼────────┐
│ Windows 10     │     │ Kali Linux        │     │ Linux Target   │
│ Target         │     │ Attacker + Agent  │     │ Persistence    │
│ Event Logs     │     │ Hydra / Bash      │     │ Cron / Files   │
└────────────────┘     └───────────────────┘     └────────────────┘
```

---

## 🚨 Detection Engineering Results

| # | Attack | MITRE | Source Log | Rule | Result |
|---|--------|-------|-----------|------|--------|
| 1 | SSH Brute Force | T1110.001 | /var/log/auth.log | 5760 | ✅ Level 10 alert |
| 2 | Sudo Escalation | T1548.004 | /var/log/auth.log | 5405 | ✅ Level 5 alert |
| 3 | Windows Scheduled Task | T1053.005 | Security EventLog | — | ❌ GAP-001 |
| 4 | Data Exfiltration Staging | T1041 | FIM (syscheck) | — | ❌ GAP-002 |
| 5 | Cron Persistence | T1053.006 | /var/log/syslog | 2833 | ✅ Level 8 alert |

**Coverage: 3/5 (60%) | Gaps documented: 2/2 | Fixes applied: 2/2**

---

## 🔍 Detection Specs (Primary Content)

Each attack has a full detection specification:

| Spec | Attack | Status |
|------|--------|--------|
| [DET-001](phase2-attack-scenarios/01-ssh-brute-force/DET-001-ssh-brute-force.md) | SSH Brute Force | ✅ Validated |
| [DET-002](phase2-attack-scenarios/02-privilege-escalation/incident-report.md) | Privilege Escalation | ✅ Validated |
| [DET-003](phase2-attack-scenarios/03-malware-persistence/incident-report.md) | Malware Persistence | ❌ Gap documented |
| [DET-004](phase2-attack-scenarios/04-data-exfiltration/incident-report.md) | Data Exfiltration | ❌ Gap documented |
| [DET-005](phase2-attack-scenarios/05-persistence-cron/DET-005-cron-persistence.md) | Cron Persistence | ✅ Validated |

Each spec contains:
- Telemetry source (exact log file)
- Detection condition (rule logic)
- Raw log evidence (real output)
- Wazuh alert JSON
- Failure cases
- Remediation

---

## ❌ Detection Gaps

### GAP-001: Windows Scheduled Task (T1053.005)

```
Attack:    Register-ScheduledTask "WindowsUpdate" -Trigger AtStartup
Expected:  Windows Security Event ID 4698
Received:  Nothing — EventChannel not configured
Root cause: ossec.conf missing Security channel subscription
```

**Fix:**
```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
  <query>Event/System[EventID=4698]</query>
</localfile>
```

→ Full analysis: [gap-analysis/DETECTION-GAP-ANALYSIS.md](phase2-attack-scenarios/gap-analysis/DETECTION-GAP-ANALYSIS.md)

---

### GAP-002: Data Exfiltration Staging (T1041)

```
Attack:    cp sensitive_files /tmp/exfil/ && tar -czf backup.tar.gz *
Expected:  FIM alert on /tmp file creation
Received:  Nothing — /tmp excluded from syscheck
Root cause: Default FIM does not monitor /tmp or /home
```

**Fix:**
```xml
<syscheck>
  <directories check_all="yes" realtime="yes">/tmp</directories>
  <directories check_all="yes" realtime="yes">/home</directories>
</syscheck>
```

→ Full analysis: [gap-analysis/DETECTION-GAP-ANALYSIS.md](phase2-attack-scenarios/gap-analysis/DETECTION-GAP-ANALYSIS.md)

---

## 📋 SIEM Query Library

Detection queries mapped to MITRE ATT&CK:

→ [detection-specs/SIEM-QUERY-LIBRARY.md](phase2-attack-scenarios/detection-specs/SIEM-QUERY-LIBRARY.md)

| Query | Use Case | Status |
|-------|----------|--------|
| QRY-001 | SSH Brute Force | ✅ Validated |
| QRY-002 | Sudo Escalation | ✅ Validated |
| QRY-003 | Cron Modification | ✅ Validated |
| QRY-004 | Win Scheduled Task | ❌ Gap (fix documented) |
| QRY-005 | File Staging /tmp | ❌ Gap (fix documented) |
| QRY-006 | Hidden Directory | 🔄 Written, pending test |
| QRY-007 | PowerShell Evasion | 🔄 Written, pending test |

---

## 📸 Evidence

Real screenshots from lab execution in `screenshots/`:

| File | Content |
|------|---------|
| 07-wazuh-cron-detection.png | Rule 2833 firing — both cron jobs detected |
| 06-cron-jobs-added.png | @reboot + */5 cron persistence verified |
| 05-backdoor-script-created.png | Malicious script + hidden directory |
| 03-archive-created.png | backup_2026_06_27.tar.gz staged |
| 01-sensitive-files-created.png | AWS keys, PII, DB credentials created |

---

## 🧰 Security Stack

| Tool | Role |
|------|------|
| Wazuh v4.14.5 | SIEM Core |
| Ubuntu 22.04 | SIEM Manager |
| Windows 10 Enterprise | Target Endpoint |
| Kali Linux | Attack Simulation + Agent |
| Hydra | Credential Attack |
| PowerShell | Windows Persistence |
| Cron / Bash | Linux Persistence |
| VirtualBox | Isolated Lab |

---

## 📊 MITRE ATT&CK Coverage

| Tactic | Technique | Coverage |
|--------|-----------|---------|
| Credential Access | T1110.001 | ✅ Detected |
| Privilege Escalation | T1548.004 | ✅ Detected |
| Persistence | T1053.006 | ✅ Detected |
| Persistence | T1053.005 | ❌ Gap |
| Collection | T1005 | ❌ Gap |
| Exfiltration | T1041 | ❌ Gap |
| Defense Evasion | T1036 | ⚠️ Partial |

---

## 🔮 Roadmap

- [ ] Retest GAP-001 and GAP-002 after fixes
- [ ] Sysmon deployment (process-level Windows telemetry)
- [ ] Sigma rule conversion for portability
- [ ] Suricata IDS integration (network-level detection)
- [ ] YARA malware detection rules
- [ ] Automated response playbooks

---

## 📬 Contact

**GitHub:** [github.com/MrBipinShrestha](https://github.com/MrBipinShrestha)  
**LinkedIn:** [linkedin.com/in/shresthabipin](https://www.linkedin.com/in/shresthabipin)  
**Location:** Sydney, Australia
