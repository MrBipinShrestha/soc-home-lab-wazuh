# Incident Report: Data Exfiltration & Archive Creation

**Report ID:** INC-2026-0604  
**Date:** June 27, 2026  
**Analyst:** SOC Team / Bipin Shrestha  
**Severity:** CRITICAL  
**Status:** Detected (Detection Gap Identified)  

---

## Executive Summary

A critical data exfiltration attack was discovered on host 192.168.100.4 (Kali Linux) on June 27, 2026 at 21:44 UTC. An attacker (root user) created and staged multiple sensitive files containing production credentials, customer PII (personally identifiable information), and database configuration, then compressed them into an archive file (backup_2026_06_27.tar.gz) for exfiltration. 

**Sensitive Data Compromised:**
- ✅ AWS access keys and secret keys
- ✅ Database credentials (admin account)
- ✅ API keys (production systems)
- ✅ Customer data (SSNs, phone numbers, emails)
- ✅ Database configuration (connection details, ports)

**Critical Finding:** The Wazuh SIEM agent did NOT detect the file staging, compression, or archive creation activities, indicating a significant detection gap in file integrity monitoring (FIM) and process monitoring capabilities.

**Data Exfiltration Statistics:**
- **Files Created:** 3 sensitive files (total 360 bytes of PII/credentials)
- **Files Staged:** 3 files copied to /tmp/exfil/ (staging area)
- **Archive Created:** backup_2026_06_27.tar.gz (450 bytes)
- **Detection Status:** NOT automatically detected by SIEM
- **Risk Level:** CRITICAL - Undetected data theft in progress

---

## Incident Timeline

| Time (UTC) | Event | Source | Details |
|-----------|-------|--------|---------|
| 21:43:00 | Sensitive files created | root user | /home/root/Documents/sensitive_data/ |
| 21:43:30 | Directory structure created | Bash shell | /home/root/Documents/sensitive_data/ |
| 21:44:00 | passwords.txt created | cat > command | 169 bytes - AWS keys, DB credentials |
| 21:44:15 | customer_data.csv created | cat > command | 102 bytes - SSN, phone, email PII |
| 21:44:30 | config.yml created | cat > command | 89 bytes - Database connection details |
| 21:45:00 | Staging directory created | mkdir -p | /tmp/exfil/ (attacker working directory) |
| 21:45:15 | Files staged for exfil | cp -r command | All 3 files copied to /tmp/exfil/ |
| 21:45:30 | **DETECTION GAP** | Wazuh SIEM | File staging NOT detected |
| 21:45:45 | Archive created | tar -czf command | backup_2026_06_27.tar.gz (450 bytes) |
| 21:46:00 | Archive ready for exfil | ls -lah | Compressed file ready for theft |
| 21:46:00+ | **DETECTION GAP** | Wazuh SIEM | Archive creation NOT detected |

---

## Attack Details

### Attack Vector: Data Staging & Compression for Exfiltration

**Method:** Manual file creation, staging, and compression  
**Target Host:** 192.168.100.4 (Kali Linux)  
**Target User:** root (privileged account)  
**Execution Context:** Shell commands (bash)  
**Exfiltration Stage:** Staging (files prepared, not yet transmitted)  

### Sensitive Data Compromised

**File 1: passwords.txt (169 bytes)**
```
Production Credentials:
database_user:admin@123
api_key:sk-proj-abc123xyz789
aws_access_key:AKIAIOSFODNN7EXAMPLE
aws_secret_key:wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

**Severity:** CRITICAL
- AWS credentials exposed (cloud infrastructure access)
- Production database credentials (data access)
- API keys (service authentication)
- Passwords in plaintext (no encryption)

**File 2: customer_data.csv (102 bytes)**
```
email,phone,ssn
john.doe@example.com,555-1234,123-45-6789
jane.smith@example.com,555-5678,987-65-4321
```

**Severity:** CRITICAL
- Customer SSNs (PII subject to GDPR, CCPA, HIPAA)
- Customer phone numbers (privacy violation)
- Customer emails (identity theft risk)
- Regulatory violation (unencrypted PII exposure)

**File 3: config.yml (89 bytes)**
```
database:
  host: prod-db.internal
  user: admin
  password: SecurePass123!
  port: 5432
```

**Severity:** CRITICAL
- Database connection details (system access)
- Admin credentials (full database access)
- Internal hostname (system reconnaissance)
- Database port information (attack mapping)

### Exfiltration Techniques

**Stage 1: File Creation**
- Files created in standard user documents location
- Demonstrates attacker system knowledge
- Uses standard file formats (txt, csv, yml)

**Stage 2: Staging**
- Files copied to /tmp/ (temporary working directory)
- Attacker consolidates stolen data
- Preparation for compression/transmission
- /tmp/ is world-writable (typical staging location)

**Stage 3: Compression**
```bash
tar -czf backup_2026_06_27.tar.gz *.txt *.csv *.yml
```

**Evasion Techniques:**
- ✅ Filename masquerades as legitimate backup (backup_2026_06_27)
- ✅ Archive format (tar.gz) compresses and obfuscates content
- ✅ Reduces file size from 360 bytes to 450 bytes (actually larger due to headers)
- ✅ Archive location: /tmp/exfil/ (non-obvious staging area)
- ✅ Ready for transmission via: SCP, FTP, HTTPS, DNS exfiltration, etc.

### MITRE ATT&CK Mapping

- **T1005** - Data from Local System (collect sensitive files)
- **T1074.001** - Staged Data (consolidate stolen data)
- **T1560.001** - Archive Collected Data (compress with tar)
- **T1537** - Transfer Data to Cloud Account (preparation for exfil)
- **T1041** - Exfiltration Over C2 Channel (next stage)

---

## Detection Gap Analysis

### CRITICAL FINDING: Undetected Data Exfiltration

**Detection Status:** ❌ **NOT DETECTED BY WAZUH**

The data staging, compression, and archive creation were **NOT detected** by Wazuh, despite:
- ✅ Wazuh agent running and active (confirmed via agent_control -lc)
- ✅ Agent file monitoring enabled
- ✅ System logs being collected

**Root Cause:** File Integrity Monitoring (FIM) and process monitoring are:
1. **Not configured for /tmp/** (temporary directory, often excluded)
2. **Not configured for /home/root/** (home directory requires explicit setup)
3. **tar/compression processes not monitored** (needs process auditing)
4. **No file creation alerts** (FIM typically monitors changes, not creation)

**What SHOULD have been detected:**
- ✅ File creation events in sensitive directories
- ✅ Copy operations (cp command) on sensitive data
- ✅ tar/compression tool execution
- ✅ Archive file creation in staging directory
- ✅ Process execution with suspicious arguments

**Why Detection Failed:**
1. **FIM Configuration Gap:** Wazuh FIM not monitoring user home directories
2. **Temporary Directory Exclusion:** /tmp/ typically excluded from monitoring
3. **Process Monitoring Gap:** No auditing of file copy/compression commands
4. **No Archive Detection Rules:** tar.gz creation not flagged as suspicious
5. **No Behavioral Analysis:** Sequential file staging pattern not detected

---

## Impact Assessment

### Confidentiality Impact: CRITICAL
- ✅ **AWS Keys Exposed:** Cloud infrastructure fully compromised
- ✅ **Database Credentials Exposed:** All customer data accessible
- ✅ **API Keys Exposed:** Third-party services compromised
- ✅ **Customer PII Exposed:** SSN, phone, email theft
- ✅ **Database Config Exposed:** Internal infrastructure reconnaissance
- **Scope:** ALL data accessible to compromised credentials

### Integrity Impact: HIGH
- **Risk:** Attacker could modify database records using stolen credentials
- **Scope:** All customer data tables
- **Evidence:** No integrity checks on staging area

### Availability Impact: HIGH
- **Risk:** Attacker could delete data using stolen credentials
- **Scope:** Production database
- **Scenario:** Ransomware or destructive attack following data theft

### Compliance Impact: CRITICAL
- **GDPR Violation:** Unencrypted customer data in plaintext, exfiltration preparation
- **CCPA Violation:** Customer SSN/phone/email exposure
- **HIPAA Violation:** Healthcare data (if applicable) in plaintext
- **PCI-DSS Violation:** Database credentials stored without encryption
- **SOC 2 Violation:** Data handling procedures not followed

### Overall Risk Level: **CRITICAL** (Active Data Breach)

---

## Immediate Response Actions

### Step 1: Secure Credentials (IMMEDIATE)

**AWS:**
```bash
aws iam list-access-keys --user-name <suspected-user>
aws iam delete-access-key --access-key-id AKIAIOSFODNN7EXAMPLE
```

**Database:**
```sql
ALTER USER admin WITH PASSWORD 'NewComplexPassword123!@#';
REVOKE ALL ON DATABASE * FROM admin;
```

**API Keys:**
- Regenerate all API keys
- Rotate keys in all services
- Check API access logs for suspicious activity

### Step 2: Contain the Breach

**On Kali (compromised system):**
```bash
# Remove exfiltration staging area
rm -rf /tmp/exfil/
rm -f /tmp/exfil/*

# Remove sensitive files
rm -rf /home/root/Documents/sensitive_data/

# Clear bash history
history -c
```

**Network:**
- Block all outbound traffic from 192.168.100.4 (prevent archive transmission)
- Monitor for any data transmission attempts
- Check DNS logs for exfiltration indicators (DNS tunneling)

### Step 3: Forensic Collection

**Preserve Evidence:**
```bash
# Copy to forensic storage BEFORE containment
dd if=/dev/sda of=/mnt/forensics/kali-backup.img
tar -czf /mnt/forensics/kali-filesystem.tar.gz /
```

**Timeline Analysis:**
- File access times: 21:43:00 - 21:46:00 UTC
- 3 minutes of data staging
- Archive ready for transmission

### Step 4: Breach Notification

**Notify:**
- Customers (PII exposed - SSN, phone, email)
- Regulators (GDPR, CCPA, HIPAA authorities)
- Payment processors (if card data involved)
- Insurance provider (cyber liability)

**Timeline:**
- GDPR: 72 hours maximum
- CCPA: Without unreasonable delay
- Breach notification laws vary by jurisdiction

---

## Root Cause Analysis

**Attack Motivation:** Data theft/exfiltration testing (Lab environment)

**Vulnerability Exploited:**
- SIEM FIM not monitoring home directories
- /tmp/ excluded from file monitoring
- No process auditing for compression tools
- No behavioral detection for data staging patterns

**Contributing Factors:**
1. **Insufficient SIEM Configuration:** FIM disabled for user directories
2. **No Process Monitoring:** tar/compression commands not logged/alerted
3. **Lack of Behavioral Analysis:** No detection of staging patterns
4. **No Data Loss Prevention (DLP):** No encryption/blocking of sensitive files
5. **No File Access Logging:** Couldn't determine which files were accessed

---

## Detection Screenshots

### Screenshot 1: Sensitive Files Created
Terminal showing creation of passwords.txt, customer_data.csv, config.yml with sensitive content:
- AWS keys, database credentials, API keys
- Customer SSN, phone, email
- Database configuration details

### Screenshot 2: File Staging & Archive Creation
Terminal showing:
- Files copied to /tmp/exfil/ staging directory
- tar -czf command compressing files into backup_2026_06_27.tar.gz
- Archive file (450 bytes) ready for exfiltration
- Complete attack chain captured

### Screenshot 3: System Activity Logs
journalctl output showing:
- System activity during attack window
- No specific file creation/access alerts
- Detection gap evident in log analysis

---

## Remediation & Recommendations

### IMMEDIATE (0-24 hours)

1. ✅ **Rotate All Compromised Credentials**
   - AWS keys, database passwords, API keys
   - Change all accounts that accessed these credentials
   - Implement new key rotation policy

2. ✅ **Breach Notification**
   - Notify customers (PII exposed)
   - File regulatory reports (GDPR, CCPA, HIPAA)
   - Contact cyber insurance provider

3. ✅ **Forensic Preservation**
   - Capture disk images before system shutdown
   - Preserve logs for investigation
   - Chain of custody documentation

4. ✅ **Network Containment**
   - Block outbound from compromised host
   - Monitor for data transmission attempts
   - Review firewall logs for suspicious connections

### SHORT-TERM (1-7 days)

**1. Enable Comprehensive File Monitoring:**

```xml
<!-- Wazuh FIM Configuration -->
<localfile>
  <location>/home/root</location>
  <log_format>command</log_format>
</localfile>

<localfile>
  <location>/tmp</location>
  <log_format>command</log_format>
</localfile>

<localfile>
  <location>/var/tmp</location>
  <log_format>command</log_format>
</localfile>
```

**2. Deploy Sysmon for Process Monitoring:**
- Monitor: tar, gzip, zip, 7z (compression)
- Monitor: cp, mv, rsync (file copy)
- Monitor: wget, curl (data transmission)
- Forward to Wazuh for correlation

**3. Create Wazuh Detection Rules:**

```xml
<!-- Archive Creation Detection -->
<rule id="70001" level="8">
  <match>tar.*-czf|zip -r|gzip|7z a</match>
  <description>Archive creation detected - possible data exfiltration</description>
</rule>

<!-- Data Staging Detection -->
<rule id="70002" level="7">
  <match>cp -r.*tmp|mv.*tmp|staging</match>
  <description>File staging to temporary directory detected</description>
</rule>

<!-- Sensitive File Detection -->
<rule id="70003" level="8">
  <match>passwords|credentials|api_key|secret_key|ssn</match>
  <description>Sensitive data file detected in logs</description>
</rule>
```

**4. Implement Data Loss Prevention (DLP):**
- Prevent unencrypted storage of sensitive data
- Block unauthorized copying of files containing PII
- Alert on SSH/SCP operations with sensitive files
- Encryption of data at rest

**5. Enable File Encryption:**
```bash
# Encrypt sensitive directories
sudo openssl enc -aes-256-cbc -salt -in passwords.txt -out passwords.txt.enc
```

### LONG-TERM (1-3 months)

**1. Data Classification & Protection:**
- Classify all data (public, internal, confidential, restricted)
- Apply encryption based on classification
- Implement access controls per classification

**2. Database Hardening:**
- Enable transparent data encryption (TDE)
- Implement column-level encryption for PII
- Enable audit logging for all access
- Implement row-level security

**3. Credential Management:**
- Implement vault solution (HashiCorp Vault, CyberArk)
- Rotate credentials every 30 days
- Audit all credential access
- Eliminate credentials from code/config files

**4. EDR (Endpoint Detection & Response):**
- Deploy EDR agent for behavioral monitoring
- Detect data exfiltration patterns
- Automatic response: block, isolate, alert

**5. Network Segmentation:**
- Isolate sensitive systems
- Whitelist outbound destinations
- DLP at network edge
- Monitor DNS for data exfiltration

---

## Compliance Impact

### Compliance Status: **VIOLATED** 🔴

| Framework | Violation | Impact |
|-----------|-----------|--------|
| **GDPR** | Unencrypted customer data exposure | €20M fine or 4% revenue |
| **CCPA** | Customer SSN/phone exposure | $100-750 per record |
| **HIPAA** | Healthcare data breach notification | $100-50,000 per violation |
| **PCI-DSS** | Database credentials in plaintext | Lose payment processing |
| **SOC 2** | Data handling procedures violated | Certification revoked |

**Required Actions:**
- Implement remediation plan above
- File breach reports with regulators
- Provide customer notification
- Implement enhanced monitoring
- Annual compliance audit

---

## Lessons Learned

**What Went Well:**
- Manual discovery identified data exfiltration in progress
- Complete evidence of staging and compression captured
- Archive file prevented from being transmitted
- Lab environment demonstrates real vulnerability

**Critical Gaps Identified:**
- SIEM FIM not monitoring home directories
- Temporary directories excluded from monitoring
- No process auditing for compression tools
- No behavioral detection for data staging
- Complete detection failure without intervention

**Recommendations for Future Testing:**
- Enable comprehensive FIM BEFORE running attacks
- Monitor all directories including /tmp and home
- Implement process auditing for suspicious tools
- Test detection rules before committing to production
- Document all detection gaps for compliance

---

## Conclusion

A critical data exfiltration attack was discovered on Kali Linux through manual investigation. The attacker successfully staged sensitive data (AWS keys, database credentials, customer PII) and compressed it into an archive file for transmission. The Wazuh SIEM failed to detect any stage of this attack, exposing a critical vulnerability in file monitoring and process auditing capabilities.

**Key Findings:**
1. ✅ Sensitive data (AWS keys, DB credentials, customer SSN) successfully stolen
2. ❌ SIEM detection completely failed
3. ⚠️ Archive created and ready for transmission (prevented by lab containment)
4. 🔴 Regulatory violations (GDPR, CCPA, HIPAA, PCI-DSS)

**Immediate Actions:**
- Rotate all compromised credentials
- Notify customers of breach
- File regulatory reports
- Preserve forensic evidence

**Recommendation:** Implement comprehensive FIM, process monitoring, and EDR as outlined above to prevent future undetected data exfiltration.

---

**Report Submitted By:** SOC Analyst  
**Date:** June 27, 2026  
**Approval:** Security Operations Manager  
