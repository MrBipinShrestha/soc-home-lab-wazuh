# Phase 2: SOC Home Lab Attack Scenarios & Incident Response

## Overview

This directory contains comprehensive documentation of 5 complete attack scenarios executed on a production-grade SOC home lab, with detailed incident reports, detection analysis, and remediation recommendations. All attacks were simulated in a controlled lab environment using Wazuh SIEM for monitoring and detection.

**Portfolio Value:** Demonstrates real-world SOC analyst skills including attack simulation, SIEM monitoring, incident documentation, detection gap analysis, and compliance understanding.

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Wazuh SIEM (Manager)                      │
│              Ubuntu 22.04 | 192.168.100.5                    │
│  ✅ Wazuh v4.14.5 | Active Agent Monitoring | Alert Rules    │
└────────────────────────────┬────────────────────────────────┘
                             │
                    NAT Network (soc-lab)
                    192.168.100.0/24
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
    ┌────▼─────┐        ┌───▼────┐        ┌────▼─────┐
    │ Windows 10│        │ Kali   │        │(Optional)│
    │(Target)   │        │(Attacker       │Metasplit │
    │192.168.   │        │& Target)       │able      │
    │100.3      │        │192.168.        │          │
    │Agent: 001 │        │100.4           │          │
    │MSEDGEWIN10│        │Agent: 004       │          │
    │✅ Active  │        │✅ Active        │          │
    └──────────┘        └────────┘        └──────────┘
```

**Infrastructure:**
- **Manager:** Ubuntu 22.04 running Wazuh v4.14.5 SIEM
- **Windows Agent:** Windows 10 Enterprise (MSEDGEWIN10) ID: 001
- **Linux Agent:** Kali Linux (dual attacker/target) ID: 004
- **Network:** NAT Network (soc-lab) with DHCP
- **All Agents:** Connected, authenticated, actively monitoring

---

## Attack Scenarios

### 1. SSH Brute Force Attack Detection
**File:** `01-ssh-brute-force/incident-report.md`

**Severity:** HIGH | **Status:** Detected ✅

**Attack Summary:**
- Tool: Hydra SSH brute force
- Target: 192.168.100.4:22 (Kali SSH)
- Attempts: 262 password attempts
- Duration: ~2 minutes
- Attacker IP: 192.168.100.4 (Kali)
- Username Targeted: root

**Detection:**
- Rule 5760 (Level 5): "sshd: authentication failed"
- Rule 5758 (Level 8): "Maximum authentication attempts exceeded"
- Detection Latency: <5 seconds
- Success Rate: 100% detection

**Evidence:**
- Hydra attack output showing password attempts
- Target auth.log showing failed logins
- Wazuh alerts showing rule triggers

**Key Learning:** Failed authentication attempts are reliably detected when properly configured.

---

### 2. Privilege Escalation Attempt
**File:** `02-privilege-escalation/incident-report.md`

**Severity:** MEDIUM | **Status:** Detected ✅

**Attack Summary:**
- Method: Unauthorized sudo command execution
- Attacker Account: testuser (UID: 1000)
- Target Privilege: root (UID: 0)
- Attempts: 7+ different escalation vectors
- Commands: whoami, visudo, passwd, su
- Duration: ~7 minutes

**Detection:**
- Rule 5405 (Level 5): "Unauthorized user attempted to use sudo"
- Triggered for each unauthorized attempt
- Detection Latency: <3 seconds per attempt
- Success Rate: 100% detection

**Evidence:**
- PowerShell script creating testuser
- auth.log showing "user NOT in sudoers"
- Multiple Wazuh alerts for each attempt

**Key Learning:** Sudo failures are detected when audit logging is enabled.

---

### 3. Malware Execution & Persistence (Scheduled Task)
**File:** `03-malware-persistence/incident-report.md`

**Severity:** CRITICAL | **Status:** Detected (Detection Gap Identified) ⚠️

**Attack Summary:**
- Method: Malicious PowerShell script + Scheduled Task
- Target: Windows 10 (192.168.100.3)
- Script: suspicious.ps1 (reconnaissance + data theft)
- Persistence: WindowsUpdate scheduled task
- Trigger: AtStartup (runs at system boot)
- Execution: Hidden PowerShell (-WindowStyle Hidden)

**Malicious Behaviors:**
- Process enumeration (Get-Process)
- Document scanning (Get-ChildItem Documents)
- Hidden execution (WindowStyle Hidden)
- Persistence across reboots

**Detection Gap:**
- ❌ Scheduled task creation NOT detected
- ❌ PowerShell execution NOT detected
- Root Cause: Windows Event Log (4698) not forwarded to SIEM
- Impact: Persistence mechanism deployed undetected

**Evidence:**
- PowerShell output showing task creation
- Task Scheduler confirming WindowsUpdate task
- Startup trigger verification

**Key Learning:** Windows event logging requires explicit configuration; many attacks go undetected without proper setup.

---

### 4. Data Exfiltration & Archive Creation
**File:** `04-data-exfiltration/incident-report.md`

**Severity:** CRITICAL | **Status:** Detected (Detection Gap Identified) ⚠️

**Attack Summary:**
- Method: File staging + compression for theft
- Target: Kali Linux (192.168.100.4)
- Sensitive Data Created:
  - passwords.txt (AWS keys, DB credentials, API keys)
  - customer_data.csv (SSNs, phone numbers, emails)
  - config.yml (database connection details)
- Files Staged: /tmp/exfil/ (attacker working directory)
- Archive Created: backup_2026_06_27.tar.gz (450 bytes)

**Data Compromised:**
- AWS access keys & secret keys
- Production database credentials
- Customer PII (SSNs, phones, emails)
- API keys for third-party services
- Database configuration details

**Detection Gap:**
- ❌ File creation NOT detected
- ❌ File staging NOT detected
- ❌ Archive creation NOT detected
- Root Cause: FIM not monitoring /tmp/ and home directories
- Impact: Sensitive data prepared for transmission, completely undetected

**Evidence:**
- Sensitive files created with realistic credentials
- Files copied to /tmp/exfil/ staging area
- tar.gz archive created and ready for exfiltration
- System logs showing activity but no SIEM alerts

**Key Learning:** File integrity monitoring (FIM) requires explicit configuration for user directories; temporary directories often excluded.

---

### 5. Persistence via Cron Job Backdoor
**File:** `05-persistence-cron/incident-report.md`

**Severity:** CRITICAL | **Status:** Detected ✅

**Attack Summary:**
- Method: Malicious cron job execution
- Target: Kali Linux (192.168.100.4)
- Backdoor Script: /tmp/update_check.sh (masked as update)
- Persistence Mechanisms:
  - @reboot /tmp/update_check.sh (runs at startup)
  - */5 * * * * /tmp/update_check.sh (runs every 5 minutes)
- Hidden Artifacts: /var/lib/.hidden/ directory + update.log

**Backdoor Capabilities:**
- Automatic execution at system boot
- Recurring execution (5-minute callback)
- Command & control callback (would reach attacker)
- Sudoers escalation capability (commented out for lab)
- Persistence survives reboots

**Detection:**
- ✅ Rule 2833 (Level 8): "Root's crontab entry changed"
- ✅ Detected both cron modifications
- Detection Latency: <5 seconds per modification
- Success Rate: 100% detection

**Why This Detection Worked:**
- Linux syslog logs cron modifications by default
- Wazuh monitors syslog for cron changes
- Rule 2833 specifically triggers on crontab modifications
- No configuration gaps (unlike scheduled tasks)

**Evidence:**
- Backdoor script creation with persistence code
- Both cron jobs verified with crontab -l
- Wazuh alerts showing immediate detection
- System logs confirming execution

**Key Learning:** Detection success depends on what's monitored. Linux cron is well-monitored by default; Windows scheduled tasks require additional configuration.

---

## Detection Summary

### What Was Detected ✅
| Attack | Rule | Detection | Latency |
|--------|------|-----------|---------|
| SSH Brute Force | 5760, 5758 | ✅ Yes | <5s |
| Privilege Escalation | 5405 | ✅ Yes | <3s |
| Cron Persistence | 2833 | ✅ Yes | <5s |

### Detection Gaps Identified ⚠️
| Attack | Issue | Root Cause | Impact |
|--------|-------|-----------|--------|
| Malware Persistence | Scheduled task not detected | Windows Event Log (4698) not forwarded | Backdoor installed undetected |
| Data Exfiltration | File operations not detected | FIM not monitoring /tmp/ and home dirs | Sensitive data staged for theft undetected |

---

## Compliance & Frameworks

### Compliance Mapping

**Attacks Successfully Detected:**
- ✅ NIST 800-53 AU.14 (Audit & Accountability)
- ✅ NIST 800-53 AC.7 (Access Control)
- ✅ PCI-DSS 10.2.5 (Logging of Access Control)
- ✅ HIPAA 164.312.b (Audit Controls)
- ✅ GDPR IV 32.2 (Monitoring & Logging)

**Detection Gaps (Compliance Violations):**
- ❌ NIST 800-53 AU-2 (Audit Events) — Scheduled task events not logged
- ❌ PCI-DSS 10.2.1 (Implement Logging) — File operations not monitored
- ❌ HIPAA 164.308(a)(5)(ii)(C) — Inadequate Windows event collection

### MITRE ATT&CK Mapping

- **T1110.001** - Brute Force: Password Guessing (SSH)
- **T1548.004** - Abuse Elevation Control Mechanism: Sudo and Sudo Caching
- **T1053.005** - Scheduled Task/Job: Cron Job (Linux) + Scheduled Task (Windows)
- **T1005** - Data from Local System (file collection)
- **T1074.001** - Staged Data (consolidate for exfiltration)
- **T1560.001** - Archive Collected Data (tar.gz compression)

---

## Key Findings & Recommendations

### Detection Gaps & Remediations

**1. Windows Scheduled Task Detection:**
- ❌ **Current:** Not detected by default
- ✅ **Fix:** Enable Event ID 4698 forwarding to SIEM
- ⏱️ **Timeline:** 1-2 hours

**2. File Integrity Monitoring (FIM):**
- ❌ **Current:** /tmp/ and home directories excluded
- ✅ **Fix:** Expand FIM configuration to all user directories
- ⏱️ **Timeline:** 2-4 hours

**3. PowerShell Logging:**
- ❌ **Current:** Not enabled
- ✅ **Fix:** Enable PowerShell script block logging
- ⏱️ **Timeline:** 1-2 hours

**4. Process Auditing:**
- ❌ **Current:** No process monitoring (need Sysmon)
- ✅ **Fix:** Deploy Sysmon on all Windows endpoints
- ⏱️ **Timeline:** 4-6 hours per endpoint

---

## Interview Talking Points

**"Tell me about your SOC lab experience:"**

> "I built a complete Wazuh-based SOC lab with Windows and Linux agents. I executed 5 realistic attack scenarios — SSH brute force, privilege escalation, malware persistence, data exfiltration, and cron backdoors. I documented each as professional incident reports with detection analysis.
> 
> The lab revealed real detection gaps: Windows scheduled task creation isn't detected by default, and file staging in /tmp/ goes undetected without proper FIM configuration. Understanding these gaps is crucial — it shows me how to recommend proper SIEM configuration based on organizational risk profile.
> 
> All 5 attacks were successful in their objectives, but 3 were automatically detected by Wazuh, while 2 required gap analysis to understand why detection failed. This demonstrates both my offensive thinking and defensive security expertise."

**"How would you improve this lab?"**

> "I'd implement:
> 1. EDR (Endpoint Detection & Response) for behavioral analysis
> 2. Sysmon on Windows for process-level visibility
> 3. PowerShell logging and script block auditing
> 4. Expand FIM to monitor all user directories and /tmp
> 5. Implement automated response (isolate systems, kill processes)
> 6. Add a SOAR (Security Orchestration) platform for automated response
> 7. Deploy honeypots to detect lateral movement
> 8. Test against MITRE ATT&CK framework systematically"

**"What did you learn from detection gaps?"**

> "That monitoring configuration is everything. The same SIEM successfully detected SSH brute force and cron persistence, but missed Windows scheduled tasks and file operations because:
> 1. Windows Event Log (4698) wasn't forwarded
> 2. FIM excluded /tmp/ and user home directories
> 3. PowerShell execution logging wasn't enabled
> 
> This taught me to approach SIEM deployment from a threat model perspective: identify what you're defending against, then explicitly configure monitoring for those attack vectors. Default configurations have massive blind spots."

---

## Files Included

```
phase2-attack-scenarios/
├── README.md (this file)
├── 01-ssh-brute-force/
│   ├── incident-report.md (detailed forensics)
│   └── screenshots/
│       ├── 01-hydra-attack.png
│       ├── 02-auth-log.png
│       └── 03-wazuh-alerts.png
├── 02-privilege-escalation/
│   ├── incident-report.md
│   └── screenshots/
│       ├── 01-escalation-attempts.png
│       ├── 02-auth-log.png
│       └── 03-wazuh-alerts.png
├── 03-malware-persistence/
│   ├── incident-report.md
│   └── screenshots/
│       ├── 01-script-creation.png
│       ├── 02-task-details.png
│       ├── 03-event-log.png
│       └── 04-detection-gap.png
├── 04-data-exfiltration/
│   ├── incident-report.md
│   └── screenshots/
│       ├── 01-file-creation.png
│       ├── 02-staging-archive.png
│       └── 03-system-logs.png
├── 05-persistence-cron/
│   ├── incident-report.md
│   └── screenshots/
│       ├── 01-backdoor-script.png
│       ├── 02-cron-jobs.png
│       └── 03-wazuh-detection.png
└── detection-rules/
    ├── ssh-brute-force.xml
    ├── privilege-escalation.xml
    ├── malware-persistence.xml
    ├── data-exfiltration.xml
    └── cron-persistence.xml
```

---

## How to Use This Portfolio

### For Interview Preparation
1. Read all 5 incident reports
2. Understand detection gaps and why they exist
3. Prepare to discuss remediation strategies
4. Be ready to explain MITRE ATT&CK mapping
5. Practice discussing compliance implications

### For Further Lab Development
1. Implement the recommended detection rule fixes
2. Add the 5th attack scenario (persistence via systemd)
3. Deploy EDR and Sysmon for process visibility
4. Implement automated response capabilities
5. Test against MITRE ATT&CK enterprise framework

### For Security Hardening
1. Review each incident report's remediation section
2. Prioritize by impact and timeline
3. Implement short-term fixes (1-7 days)
4. Plan long-term infrastructure improvements
5. Measure effectiveness with re-execution

---

## Skills Demonstrated

✅ **SIEM Administration:** Wazuh deployment, agent configuration, rule creation  
✅ **Attack Simulation:** Real attack tools (Hydra, PowerShell, cron, tar)  
✅ **Log Analysis:** auth.log, Event Viewer, syslog interpretation  
✅ **Incident Documentation:** Professional reports with timeline, impact, recommendations  
✅ **Detection Engineering:** Understanding of detection rules, gaps, and evasion  
✅ **Compliance Knowledge:** NIST 800-53, PCI-DSS, HIPAA, GDPR mapping  
✅ **Security Hardening:** Recommendations for defensive improvements  
✅ **Both Offense & Defense:** Understanding attacker perspective + defender capabilities  

---

## Next Steps

### Immediate
- [ ] Review all 5 incident reports
- [ ] Understand why each attack succeeded
- [ ] Understand why 2 attacks weren't detected
- [ ] Prepare interview answers

### Short-term (1-2 weeks)
- [ ] Implement Windows Event Log forwarding
- [ ] Expand FIM to user directories
- [ ] Enable PowerShell logging
- [ ] Test detection with re-execution

### Long-term (1-3 months)
- [ ] Deploy Sysmon on Windows
- [ ] Deploy EDR (CrowdStrike, Microsoft Defender ATP)
- [ ] Implement SOAR for automated response
- [ ] Test against MITRE ATT&CK framework
- [ ] Add 2-3 more attack scenarios

---

## Contact & Questions

For questions about this lab setup:
- Review individual incident reports
- Check detection rule recommendations
- Refer to remediation sections for implementation details

For interviews:
- Understand the 5 attacks thoroughly
- Be prepared to discuss detection gaps
- Explain why certain attacks succeeded/failed
- Show your defensive security mindset

---

**Created:** June 27, 2026  
**Lab Duration:** ~4 hours of active testing  
**Total Documentation:** 25,000+ words  
**Incident Reports:** 5 comprehensive reports  
**Attack Scenarios:** 5 successful simulations  
**Detection Success Rate:** 60% (3/5 automatically detected)  

**Status:** ✅ Production-Ready Portfolio Content
