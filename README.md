# 🛡️ SOC Detection Engineering Lab — Wazuh SIEM

![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-blue)
![MITRE ATT&CK](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-red)
![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Windows-green)
![Security](https://img.shields.io/badge/Focus-SOC%20Detection%20Engineering-orange)

A hands-on **Security Operations Center (SOC) laboratory** designed to simulate real-world cyber attacks, validate SIEM detections, investigate alerts, and document incident response workflows.

This project demonstrates an end-to-end blue-team workflow:

```
Attack Simulation
        ↓
Log Collection
        ↓
SIEM Detection
        ↓
Alert Investigation
        ↓
MITRE ATT&CK Mapping
        ↓
Incident Reporting
        ↓
Detection Improvement
```

---

# 🎯 Project Objective

The objective of this lab is to build practical experience in:

- SOC operations
- SIEM deployment and monitoring
- Detection engineering
- Endpoint security monitoring
- Incident response
- MITRE ATT&CK mapping
- Security alert investigation

The lab focuses on validating whether security controls can detect realistic attacker behaviour and identifying areas where detection coverage can be improved.

---

# 🏗️ Lab Architecture

## Environment

| Component | Role |
|-----------|------|
| Ubuntu Server | Wazuh Manager |
| Windows Endpoint | Agent Monitoring |
| Linux Endpoint | Agent Monitoring |
| Kali Linux | Attack Simulation |
| Wazuh Dashboard | Security Monitoring |

Architecture:

```
                 Kali Linux
              Attack Simulation
                    |
                    |
        ---------------------------
        |                         |
     Windows                  Linux
      Agent                  Agent
        |                         |
        ---------------------------
                    |
              Wazuh Manager
                    |
            Security Dashboard
                    |
             Alert Investigation
```

---

# 🔧 Technology Stack

## SIEM & Monitoring

- Wazuh SIEM
- Wazuh Agents
- Security Dashboard
- Log Analysis

## Operating Systems

- Ubuntu Server
- Windows
- Linux
- Kali Linux

## Security Frameworks

- MITRE ATT&CK
- Incident Response Lifecycle

## Networking

- NAT Networking
- SSH
- Endpoint Communication

---

# 🚨 Attack Scenarios Tested

The lab contains five controlled attack simulations.

| Attack Scenario | MITRE ATT&CK | Detection Status |
|---|---|---|
| SSH Brute Force | T1110 - Brute Force | ✅ Detected |
| Privilege Escalation | T1068 - Exploitation for Privilege Escalation | ✅ Detected |
| Malware Persistence | T1547 - Boot or Logon Autostart Execution | ✅ Detected |
| Data Exfiltration | T1041 - Exfiltration Over C2 Channel | ⚠️ Detection Gap Identified |
| Cron Backdoor | T1053 - Scheduled Task/Job | ⚠️ Detection Gap Identified |

---

# 🔍 Detection Engineering Workflow

Each attack scenario follows the same SOC investigation process:

## 1. Attack Simulation

Generate controlled malicious activity in the lab environment.

Examples:

- Password attacks
- Suspicious process execution
- Persistence techniques
- File modification
- Network activity

---

## 2. Log Collection

Wazuh collects:

- Authentication logs
- System events
- Process activity
- File integrity changes
- Network-related events

---

## 3. Alert Investigation

Analyst workflow:

```
Alert Generated
       ↓
Review Event Details
       ↓
Identify Indicators
       ↓
Analyse Timeline
       ↓
Determine Severity
       ↓
Document Findings
```

---

## 4. MITRE ATT&CK Mapping

Each detection is mapped to attacker behaviour:

Example:

```
Attack:
SSH Brute Force

MITRE Technique:
T1110 - Brute Force

Evidence:
Failed authentication attempts

Response:
Block source IP
Investigate account activity
```

---

# 📊 Detection Results

## Detection Coverage

| Metric | Result |
|---|---|
| Attack Scenarios Tested | 5 |
| Automatically Detected | 3 |
| Detection Gaps Identified | 2 |
| Incident Reports Created | 5 |

The goal was not only to achieve successful detections but also to identify limitations and improve security monitoring capability.

---

# 📁 Incident Response Reports

Each attack scenario includes investigation documentation:

| Report | Investigation |
|---|---|
| IR-001 | SSH Brute Force Investigation |
| IR-002 | Privilege Escalation Analysis |
| IR-003 | Malware Persistence Investigation |
| IR-004 | Data Exfiltration Analysis |
| IR-005 | Cron Backdoor Investigation |

Each report contains:

- Attack description
- Timeline analysis
- Evidence collected
- Detection results
- MITRE mapping
- Recommended remediation

---

# 📸 Screenshots

The project includes evidence from:

- Wazuh Dashboard alerts
- Security events
- Agent monitoring
- Attack execution
- Investigation results

```
screenshots/

├── dashboard/
├── alerts/
├── attacks/
└── investigations/
```

---

# 🧪 Example Investigation

## SSH Brute Force Detection

### Attack

Multiple failed SSH login attempts were generated against a monitored endpoint.

### Detection

Wazuh identified abnormal authentication behaviour.

### Investigation

Analysed:

- Source IP
- Username attempts
- Authentication failures
- Event timeline

### Response

Recommended:

- Block malicious source
- Enable stronger authentication
- Monitor repeated attempts

MITRE ATT&CK:

```
T1110 - Brute Force
```

---

# 🧠 Security Lessons Learned

## Successful Detection Areas

✅ Authentication attacks  
✅ Endpoint monitoring  
✅ Suspicious activity analysis  
✅ Log-based investigation  

## Improvement Areas

⚠️ Advanced exfiltration detection  
⚠️ Additional custom detection rules  

Future improvements:

- Sigma rules
- Custom Wazuh rules
- Threat intelligence integration
- SOAR automation

---

# 🚀 Future Enhancements

Planned improvements:

- Add custom Wazuh detection rules
- Integrate Sigma rules
- Add TheHive incident management
- Add Shuffle SOAR automation
- Integrate threat intelligence feeds
- Expand MITRE ATT&CK coverage

---

# 🎓 Skills Demonstrated

This project demonstrates practical knowledge of:

- SOC Operations
- SIEM Administration
- Detection Engineering
- Incident Response
- Threat Investigation
- Log Analysis
- MITRE ATT&CK Framework
- Linux Security
- Windows Security
- Blue Team Methodology

---

# 👤 Author

## Bipin Shrestha

Cybersecurity Student | SOC Analyst | Detection Engineering

📍 Sydney, Australia 🇦🇺

GitHub:
https://github.com/MrBipinShrestha

LinkedIn:
https://www.linkedin.com/in/shresthabipin/

---

⭐ If you find this project useful, consider starring the repository.
