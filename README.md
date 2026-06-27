# 🛡️ SOC Detection Engineering Lab — Wazuh SIEM

![Wazuh](https://img.shields.io/badge/Wazuh-v4.14.5-blue?style=for-the-badge)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red?style=for-the-badge)
![Security Focus](https://img.shields.io/badge/Focus-Detection_Engineering-purple?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)

> **Enterprise-grade SOC simulation environment focused on detection engineering, adversary emulation, and SIEM rule development using Wazuh.**
>
> This lab replicates real-world attacker behavior, validates detection coverage, and identifies telemetry gaps in endpoint + network visibility.

---

## 🧠 Security Engineering Objective

This project is designed to answer a real SOC question:

> "Can we reliably detect attacker behavior across identity, endpoint, and persistence layers using SIEM telemetry alone?"

To validate this, I built an end-to-end attack simulation environment and measured:

- Detection coverage
- Rule effectiveness
- Logging gaps
- MITRE ATT&CK mapping accuracy

---

## 🏗️ SOC Architecture

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
┌───▼────────────┐     ┌───────▼───────────┐     ┌───────▼───────────┐
│ Windows 10     │     │ Kali Linux        │     │ Linux Endpoint    │
│ Target Endpoint│     │ Attacker + Agent  │     │ Persistence Host  │
│ Event Logs     │     │ Hydra / Simulation│     │ Cron / File Mods  │
└────────────────┘     └───────────────────┘     └───────────────────┘
```

---

## 🚨 Detection Engineering Outcomes

### Attack Simulation Coverage

| # | Attack Scenario | MITRE Technique | Detection Status | Outcome |
|---|----------------|-----------------|-----------------|---------|
| 1 | SSH Brute Force | T1110.001 | ✅ Detected | High-confidence alert |
| 2 | Privilege Escalation (sudo abuse) | T1548.004 | ✅ Detected | Rule-based detection |
| 3 | Windows Persistence (Scheduled Task) | T1053.005 | ❌ Missed | Logging gap identified |
| 4 | Data Exfiltration (staging) | T1041 / T1560 | ❌ Missed | FIM coverage issue |
| 5 | Cron-based Persistence | T1053.006 | ✅ Detected | Behavioral match |

### 📊 Detection Coverage Score: **60%**

---

## 🔍 Detection Engineering Insights

### 🟢 Successfully Detected Behaviors

**SSH Brute Force (T1110.001)**
- Correlated failed login attempts from single IP
- Triggered Wazuh rule 5760
- High-confidence brute force pattern detection

**Privilege Escalation Attempt (T1548.004)**
- Detection of unauthorized sudo execution
- Matched user privilege boundary violation
- Triggered Wazuh rule 5405

**Cron Persistence (T1053.006)**
- File integrity monitoring detected crontab modifications
- Real-time alerting enabled via rule 2833
- <5 second detection latency

---

### 🔴 Detection Gaps Identified

**1. Windows Scheduled Task Abuse (T1053.005)**

Issue: No telemetry from Event ID 4698

Root Cause: Missing Windows EventChannel ingestion

Fix:
```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
  <query>Event/System[EventID=4698]</query>
</localfile>
```

**2. Data Exfiltration Simulation (T1041)**

Issue: File staging activity not detected

Root Cause: Insufficient File Integrity Monitoring scope

Fix:
```xml
<syscheck>
  <directories check_all="yes">/tmp</directories>
  <directories check_all="yes">/home</directories>
  <directories check_all="yes">/root</directories>
</syscheck>
```

---

## 📊 MITRE ATT&CK Coverage

| Tactic | Technique | Detection Scenario |
|--------|----------|--------------------|
| Credential Access | T1110.001 | SSH brute force |
| Privilege Escalation | T1548.004 | sudo abuse detection |
| Persistence | T1053.005 | Windows scheduled task |
| Persistence | T1053.006 | Cron job modification |
| Collection | T1005 | Data staging |
| Exfiltration | T1041 | Archive creation |
| Defense Evasion | T1036 | Hidden file execution |

---

## 🧪 Repository Structure

```
soc-home-lab-wazuh/
├── phase2-attack-scenarios/
│   ├── 01-ssh-brute-force/
│   │   └── incident-report.md
│   ├── 02-privilege-escalation/
│   │   └── incident-report.md
│   ├── 03-malware-persistence/
│   │   └── incident-report.md
│   ├── 04-data-exfiltration/
│   │   └── incident-report.md
│   └── 05-persistence-cron/
│       └── incident-report.md
├── screenshots/
│   ├── 07-wazuh-cron-detection.png
│   ├── 06-cron-jobs-added.png
│   ├── 05-backdoor-script-created.png
│   ├── 03-archive-created.png
│   └── 01-sensitive-files-created.png
└── lab-setup/
```

---

## 📸 Evidence (SOC Validation Artifacts)

All detections backed by:

- Real-time Wazuh alert logs
- SIEM dashboard screenshots
- Attack execution traces
- Rule firing validation
- MITRE ATT&CK mapping evidence

Screenshots in `screenshots/` folder — real evidence from live lab execution.

---

## 🧰 Security Stack

| Tool | Role |
|------|------|
| Wazuh v4.14.5 | SIEM Core |
| Ubuntu 22.04 | SIEM Manager Host |
| Windows 10 Enterprise | Target Endpoint |
| Kali Linux | Attack Simulation + Agent |
| Hydra | Credential Attack Simulation |
| PowerShell | Windows Persistence Simulation |
| Cron / Bash | Linux Persistence Simulation |
| VirtualBox | Isolated SOC Environment |

---

## 📈 Engineering Outcomes

This lab demonstrates:

- End-to-end SOC workflow: **Attack → Detection → Investigation → Gap Analysis**
- Detection engineering mindset (not just log monitoring)
- SIEM rule tuning and validation
- MITRE ATT&CK-based visibility mapping
- Real-world SOC gap identification and remediation

---

## 🔮 Roadmap

- [ ] Sysmon integration (process-level telemetry)
- [ ] Sigma rule conversion for portable detections
- [ ] Suricata IDS integration (network visibility)
- [ ] YARA-based malware detection rules
- [ ] Automated incident response playbooks
- [ ] Velociraptor endpoint forensics integration

---

## 📬 Contact

**GitHub:** [github.com/MrBipinShrestha](https://github.com/MrBipinShrestha)  
**LinkedIn:** [linkedin.com/in/bipinshrestha](https://www.linkedin.com/in/bipinshrestha)  
**Location:** Sydney, Australia
