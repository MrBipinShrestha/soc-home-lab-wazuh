# 🚨 SOC Detection Engineering Lab — Wazuh SIEM

<p align="center">

<img src="https://img.shields.io/badge/SOC-Detection%20Engineering-blue"/>
<img src="https://img.shields.io/badge/SIEM-Wazuh-orange"/>
<img src="https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-red"/>
<img src="https://img.shields.io/badge/Focus-Blue%20Team-green"/>

</p>


# 🕵️ Incident Scenario

## "A suspicious attacker is targeting our network..."

Your mission as a SOC analyst:

> Detect the attack. Investigate the evidence. Understand the attacker behaviour. Improve detection coverage.

A controlled adversary simulation was performed against a monitored environment.

The SOC pipeline:

```
Attacker Activity
        |
        ↓
Endpoint Events
        |
        ↓
Wazuh SIEM
        |
        ↓
Alert Generation
        |
        ↓
SOC Investigation
        |
        ↓
Incident Response
        |
        ↓
Detection Improvement
```

---

# 🎯 Project Overview

This project is a hands-on **Security Operations Center (SOC) laboratory** built to simulate realistic cyber attacks and validate defensive monitoring capabilities.

The objective is not only to generate alerts, but to follow a real SOC workflow:

✅ Attack simulation  
✅ Detection validation  
✅ Evidence collection  
✅ Threat investigation  
✅ MITRE ATT&CK mapping  
✅ Incident documentation  
✅ Detection improvement  


---

# 🏗️ Lab Architecture


```
                         ATTACKER

                       Kali Linux
                    Attack Simulation

                            |
                            |
        -----------------------------------------
        |                                       |
        ↓                                       ↓

 Windows Endpoint                  Linux Endpoint
    Agent                              Agent

        |                                       |

        -----------------------------------------

                         ↓

                  Wazuh Manager

                         ↓

                Security Dashboard

                         ↓

              SOC Analyst Investigation

```


## Environment

| Component | Purpose |
|-|-|
| Kali Linux | Attack simulation |
| Ubuntu Server | Wazuh Manager |
| Windows | Endpoint monitoring |
| Linux | Endpoint monitoring |
| Wazuh Dashboard | Detection & investigation |

---

# 🚨 Attack Simulations

The lab simulates real attacker behaviour mapped to MITRE ATT&CK.


| Attack | MITRE Technique | Result |
|-|-|-|
| SSH Brute Force | T1110 Brute Force | ✅ Detected |
| Privilege Escalation | T1068 | ✅ Detected |
| Malware Persistence | T1547 | ✅ Detected |
| Data Exfiltration | T1041 | ⚠️ Detection Gap |
| Cron Backdoor | T1053 | ⚠️ Detection Gap |


---

# 📊 SOC Detection Scorecard


```
Detection Coverage

████████████░░░░░░░░

3 / 5 Attack Scenarios Automatically Detected

60% Initial Coverage

```

The purpose of this project was not to show perfect detection.

Real SOC environments have visibility gaps.

The important skill is:

> Identify gaps → Improve controls → Increase detection capability.


---

# 🔍 Investigation Walkthrough


## Case 001 — SSH Brute Force


### Scenario

An attacker attempts repeated authentication attempts against a monitored Linux endpoint.


### Attack Behaviour

```
Multiple failed SSH login attempts

↓

Suspicious authentication pattern

↓

Security event generated
```


### SOC Investigation


Analyst reviews:

```
✓ Source IP
✓ Username attempts
✓ Authentication timeline
✓ Failed login frequency
✓ Host activity
```


### MITRE ATT&CK

```
T1110
Brute Force
```


### Response Actions

- Investigate source IP
- Review affected account
- Recommend blocking malicious source
- Improve authentication controls


---

# 🧩 MITRE ATT&CK Coverage


| Technique | ID | Category |
|-|-|-|
| Brute Force | T1110 | Credential Access |
| Privilege Escalation | T1068 | Privilege Escalation |
| Boot/Logon Persistence | T1547 | Persistence |
| Scheduled Task | T1053 | Persistence |
| Exfiltration | T1041 | Command & Control |


---

# 🖥️ Detection Evidence


## Wazuh Dashboard

Add screenshots:

```
screenshots/

├── dashboard.png
├── ssh-alert.png
├── malware-alert.png
├── investigation.png

```


Example:


```
Alert Generated

Rule:
SSH Multiple Failed Login Attempts

Severity:
High

Agent:
Linux Endpoint

MITRE:
T1110

Status:
Investigating

```


---

# 📁 Incident Response Reports


Each attack has a dedicated investigation report.


```
incidents/

├── IR-001-SSH-Bruteforce.md

├── IR-002-Privilege-Escalation.md

├── IR-003-Malware-Persistence.md

├── IR-004-Data-Exfiltration.md

└── IR-005-Cron-Backdoor.md

```


Each report contains:


- Incident summary
- Attack timeline
- Evidence collected
- Detection analysis
- MITRE mapping
- Recommended remediation


---

# 🧠 Detection Engineering Lessons


## What Worked

✅ Authentication monitoring  
✅ Endpoint visibility  
✅ Log-based detection  
✅ MITRE mapping  


## Detection Gaps Identified


⚠️ Advanced exfiltration behaviour

Improvement:

- Network monitoring
- Custom detection rules
- Threat intelligence enrichment


⚠️ Scheduled task visibility

Improvement:

- Custom Wazuh rules
- Additional endpoint telemetry


---

# 🛠️ Technology Stack


## SIEM

- Wazuh


## Operating Systems

- Linux
- Windows
- Ubuntu Server


## Security Framework

- MITRE ATT&CK


## Analysis

- Log Analysis
- Incident Response
- Threat Detection


## Networking

- SSH
- NAT Networking
- Endpoint Communication


---

# 🚀 Future SOC Upgrades


Planned evolution:


```
Current Lab

Wazuh SIEM

        ↓

Custom Detection Rules

        ↓

Sigma Rules

        ↓

Threat Intelligence

        ↓

TheHive Case Management

        ↓

Shuffle SOAR Automation

        ↓

Mini Enterprise SOC Platform

```


---

# 🎓 Skills Demonstrated


🛡️ SOC Operations  
🔍 Threat Investigation  
📊 SIEM Monitoring  
🚨 Alert Analysis  
🧩 MITRE ATT&CK Mapping  
🐧 Linux Security  
🪟 Windows Security  
⚔️ Attack Simulation  
📄 Incident Reporting  
🛠️ Detection Engineering  


---

# 👨‍💻 Author


## Bipin Shrestha

Cybersecurity Student | SOC Analyst | Detection Engineering


📍 Sydney, Australia 🇦🇺


GitHub:

https://github.com/MrBipinShrestha


LinkedIn:

https://www.linkedin.com/in/shresthabipin/


---

# ⭐ Why This Project Matters


Most beginners demonstrate:

> "I installed Wazuh."


This project demonstrates:

> "I simulated attacks, analysed detections, documented incidents, identified gaps, and improved security monitoring."


That is the mindset required in a real SOC environment.


⭐ Star this repository if you find it useful.
