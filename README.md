# 🛡️ SOC Home Lab: Wazuh SIEM

**SIEM Administration • Incident Response • Detection Engineering • MITRE ATT&CK**

![Wazuh](https://img.shields.io/badge/Wazuh-v4.14.5-blue?style=for-the-badge)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK-red?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Production%20Ready-success?style=for-the-badge)

A complete Security Operations Center (SOC) home lab demonstrating SIEM administration, attack simulation, incident response, and detection engineering using Wazuh.

## 🎯 Overview

This lab showcases practical SOC analyst skills through:
- **5 real attack scenarios** executed in a controlled environment
- **Professional incident response documentation** with detailed analysis
- **Detection engineering** including identification and remediation of security gaps
- **Log analysis and correlation** across Windows and Linux systems
- **Compliance mapping** to industry frameworks (NIST 800-53, PCI-DSS, HIPAA, GDPR)

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────┐
│   Wazuh SIEM Manager (Ubuntu)   │
│   192.168.100.5                 │
└──────────────┬──────────────────┘
               │
        NAT Network (soc-lab)
      192.168.100.0/24
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼──┐  ┌───▼──┐  ┌────▼───┐
│Win10 │  │ Kali │  │Optional │
│Agent │  │Agent │  │ Target  │
│ID:001│  │ID:004│  │         │
└──────┘  └──────┘  └─────────┘
✅ Connected
```

---

## 🔴 Attack Scenarios

| # | Scenario | Technique | Detection | Report |
|---|----------|-----------|-----------|--------|
| 1 | SSH Brute Force | T1110.001 | ✅ Detected | [View](phase2-attack-scenarios/01-ssh-brute-force/) |
| 2 | Privilege Escalation | T1548.004 | ✅ Detected | [View](phase2-attack-scenarios/02-privilege-escalation/) |
| 3 | Malware Persistence | T1053.005 | ⚠️ Detection Gap | [View](phase2-attack-scenarios/03-malware-persistence/) |
| 4 | Data Exfiltration | T1041 | ⚠️ Detection Gap | [View](phase2-attack-scenarios/04-data-exfiltration/) |
| 5 | Cron Job Backdoor | T1053.006 | ✅ Detected | [View](phase2-attack-scenarios/05-persistence-cron/) |

---

## 🎓 Key Findings

### Detection Success: 60% (3 of 5 automatically detected)

**What Was Detected:**
- ✅ SSH brute force attacks (Rule 5760, 5758)
- ✅ Privilege escalation attempts (Rule 5405)
- ✅ Cron job modifications (Rule 2833)

**Detection Gaps Identified:**
- ❌ Windows scheduled task creation (Event ID 4698 not forwarded)
- ❌ File staging operations (FIM not monitoring /tmp/ and home directories)

**Why This Matters:**
Understanding detection gaps is crucial for SOC analysts. This lab shows:
1. What's detected by default (robust monitoring)
2. What requires configuration (SIEM tuning)
3. Why gaps exist (architectural decisions)
4. How to remediate (concrete recommendations)

---

## 📊 Incident Reports

Each scenario includes:
- **Attack timeline** with precise timestamps
- **Forensic analysis** of artifacts and evidence
- **Detection analysis** (what worked, what didn't, why)
- **Impact assessment** (confidentiality, integrity, availability)
- **Remediation recommendations** (immediate, short-term, long-term)
- **Compliance implications** (NIST, PCI-DSS, HIPAA, GDPR)

Total analysis: **Professional incident documentation** covering complete attack lifecycle.

---

## 🛠️ Skills Demonstrated

| Category | Skills |
|----------|--------|
| **SIEM** | Wazuh administration, agent deployment, rule creation |
| **Detection Engineering** | Rule tuning, log analysis, alert correlation |
| **Attack Simulation** | Hydra, PowerShell, cron, tar archives |
| **Log Analysis** | auth.log, Event Viewer, syslog interpretation |
| **Incident Response** | Timeline reconstruction, evidence collection, reporting |
| **Linux/Windows** | OS hardening, event logging, audit configuration |
| **Compliance** | NIST 800-53, PCI-DSS, HIPAA, GDPR mapping |
| **Threat Framework** | MITRE ATT&CK correlation and analysis |

---

## 📁 Repository Structure

```
soc-home-lab-wazuh/
├── README.md (this file)
├── phase2-attack-scenarios/
│   ├── README.md (comprehensive guide)
│   ├── 01-ssh-brute-force/
│   │   ├── incident-report.md
│   │   └── screenshots/
│   ├── 02-privilege-escalation/
│   ├── 03-malware-persistence/
│   ├── 04-data-exfiltration/
│   ├── 05-persistence-cron/
│   └── detection-rules/
│       └── (Wazuh rule examples)
├── screenshots/
│   ├── wazuh-dashboard.png
│   ├── agent-status.png
│   ├── alert-example.png
│   └── incident-timeline.png
└── lab-setup/
    ├── installation-guide.md
    └── architecture-diagram.md
```

---

## 🎯 MITRE ATT&CK Coverage

This lab systematically covers multiple attack techniques:

| Tactic | Technique | Attack Scenario |
|--------|-----------|-----------------|
| **Credential Access** | T1110.001 (Brute Force) | SSH Brute Force |
| **Privilege Escalation** | T1548.004 (Sudo/Sudo Caching) | Privilege Escalation |
| **Persistence** | T1053.005 (Scheduled Task) | Malware Persistence |
| **Persistence** | T1053.006 (Cron) | Cron Backdoor |
| **Collection** | T1005 (Data from Local System) | Data Exfiltration |
| **Exfiltration** | T1041 (Exfil Over C2) | Archive Staging |

---

## 🔍 Detection Gap Analysis

### Why 2 Attacks Weren't Detected

This isn't a failure — it's an insight.

**Attack 3 (Scheduled Task):**
- Windows Event ID 4698 requires explicit forwarding
- Many organizations don't configure this
- Remediation: Enable Event Log subscription in Wazuh

**Attack 4 (Data Exfiltration):**
- File Integrity Monitoring (FIM) excluded /tmp/ by default
- User home directories often not monitored
- Remediation: Expand FIM coverage

**Why This Matters for SOC Analysts:**
- Real-world SIEM deployments have gaps
- Understanding *why* gaps exist is valuable
- Knowing how to remediate is critical
- Showing both detection successes AND gaps demonstrates maturity

---

## 💡 Key Insights

### What This Lab Proves

✅ **Proper SIEM configuration works** — 3 attacks detected with <5s latency  
✅ **Configuration gaps are common** — 2 attacks missed due to default settings  
✅ **Detection is tunable** — All gaps have concrete solutions  
✅ **Log analysis requires expertise** — Understanding *what* to monitor matters  

### Interview Talking Points

> "I built a Wazuh SIEM lab and executed 5 realistic attacks. Three were automatically detected, but two weren't — which actually taught me more. The scheduled task creation and data staging attacks revealed why many organizations have blind spots: missing event log forwarding and incomplete FIM configuration. I documented not just what happened, but why the detection gaps existed and how to fix them."

---

## 📋 Getting Started

### For Security Professionals
1. Read the [Phase 2 comprehensive guide](phase2-attack-scenarios/README.md)
2. Review incident reports 1-5 in order
3. Study detection analysis and gaps
4. Reference remediation timelines

### For Interview Preparation
1. Understand all 5 attack scenarios
2. Know why 3 were detected and 2 weren't
3. Be able to discuss detection gap remediation
4. Prepare to explain MITRE ATT&CK correlation

### For Further Development
- Implement recommended detection rule improvements
- Deploy Sysmon for enhanced process monitoring
- Add EDR solution for behavioral detection
- Test automated response capabilities

---

## 🎓 Compliance Mapping

**Attacks Detected Successfully:**
- ✅ NIST 800-53 SI-4 (Information System Monitoring)
- ✅ PCI-DSS 10.2.5 (Monitoring of Access to Cardholder Data)
- ✅ HIPAA 164.312.b (Audit Controls)

**Detection Gaps (Require Remediation):**
- ❌ NIST 800-53 AU-2 (Audit Events) — scheduled task logging
- ❌ PCI-DSS 10.2.1 (Implement Logging) — file monitoring gaps
- ❌ HIPAA 164.308(a)(5)(ii)(C) — comprehensive event collection

---

## 🚀 Deployment Details

**Manager:**
- Wazuh v4.14.5 on Ubuntu 22.04
- Deployed via OVA (4GB RAM)
- Agents enrolled and authenticated

**Windows Agent:**
- Windows 10 Enterprise Evaluation
- Wazuh Agent ID: 001
- Connected and sending logs

**Linux Agent:**
- Kali Linux
- Wazuh Agent ID: 004
- Dual role: attacker and monitoring target

---

## 📊 Lab Statistics

| Metric | Value |
|--------|-------|
| Attack Scenarios | 5 |
| Detection Success Rate | 60% (3/5) |
| Detection Latency | <5 seconds |
| Incident Reports | 5 |
| Total Analysis | Professional documentation |
| Lab Runtime | 4+ hours |

---

## 🔗 Related Projects

**On the same GitHub:**
- **network-anomaly-detection-unsw-nb15** — Machine learning approach to intrusion detection (MIT516)
- **clamav-malware-detection-lab** — Antivirus and malware analysis
- **phishing-attack-lab-kit** — Social engineering and phishing simulation
- **social-engineering-awareness-lab** — Human security testing

---

## 💼 What This Shows Employers

This repository demonstrates practical skills employers actually need:

• **SIEM administration** — Wazuh deployment, configuration, agent management
• **Incident response** — Professional documentation, timeline analysis, remediation
• **Detection engineering** — Rule tuning, alert correlation, gap identification
• **Log analysis** — Windows and Linux log interpretation
• **MITRE ATT&CK** — Framework application to real attacks
• **Security analysis** — Both what works AND why detection fails
• **Compliance** — NIST 800-53, PCI-DSS, HIPAA mapping

**Best for:** SOC Analyst, Detection Engineer, Security Operations Analyst, Junior Blue Team roles

---

## 📖 How to Use This Repository

**Read in this order:**
1. This README (overview)
2. [Phase 2 Guide](phase2-attack-scenarios/README.md) (detailed context)
3. Individual incident reports (1-5)
4. Detection rules directory

**For each incident report, review:**
- Executive summary
- Attack timeline
- Detection analysis
- Remediation recommendations

---

## 🚀 Future Enhancements

Planned improvements:
- [ ] Sysmon deployment for enhanced Windows process monitoring
- [ ] Sigma rule conversion for SIEM portability
- [ ] YARA malware detection rules
- [ ] Suricata IDS/IPS integration
- [ ] Velociraptor endpoint forensics
- [ ] Automated response playbooks
- [ ] Additional attack scenarios (lateral movement, privilege abuse, data theft)

---

## 📸 Lab Screenshots

Real evidence from attack execution (see `screenshots/` folder):
- Sensitive data creation and staging
- Archive compression for exfiltration
- System activity logs during attacks
- Wazuh detection alerts (Rule 2833, 5760, 5405)
- Agent status and dashboard

---

## ✨ Highlights

What makes this lab valuable:

1. **Real attacks, not simulations** — Actual tools (Hydra, PowerShell, cron)
2. **Professional documentation** — Incident reports suitable for compliance
3. **Honest about gaps** — Showing what *didn't* work is more valuable than hiding it
4. **Actionable remediation** — Not just identifying problems, but solving them
5. **Compliance-aware** — Mapped to industry frameworks
6. **MITRE ATT&CK aligned** — Modern threat framework correlation

---

## 📧 Questions or Feedback

For questions about the lab setup, attack scenarios, or detection engineering:
- Review the comprehensive guides in each scenario directory
- Check incident report remediation sections
- Reference detection rule recommendations

---

## 📜 License

MIT License — See LICENSE file

---

**Created:** June 27, 2026  
**Status:** ✅ Production-Ready Portfolio Content  

**Suitable for:**
- Job applications (SOC/Detection roles)
- Interview preparation
- Portfolio demonstration
- Further security research
- Homelab reference
