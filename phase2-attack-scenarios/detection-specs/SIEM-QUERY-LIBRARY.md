# SIEM Query Library

**SIEM:** Wazuh v4.14.5  
**Lab:** SOC Detection Engineering Lab  
**Analyst:** Bipin Shrestha  

---

## Query Format

Each query includes:
- **Use case** — what this detects
- **Data source** — where the log comes from
- **Query** — filter/search syntax
- **Alert threshold** — when to fire
- **MITRE mapping** — technique coverage

---

## QRY-001: SSH Brute Force Detection

**Use case:** Detect repeated SSH login failures from single IP

**Data source:** `/var/log/auth.log` via Wazuh Agent

**Wazuh Rule:**
```xml
<rule id="5760" level="10">
  <if_matched_sid>5700</if_matched_sid>
  <same_source_ip/>
  <description>sshd: brute force detected</description>
  <mitre><id>T1110</id></mitre>
</rule>
```

**Threshold:** 5 failures within 60 seconds from same IP

**MITRE:** T1110.001 (Brute Force: Password Guessing)

**Test command:**
```bash
hydra -l root -P wordlist.txt ssh://TARGET -t 4
```

**Expected alert level:** 10 (Critical)

---

## QRY-002: Privilege Escalation via Sudo

**Use case:** Detect unauthorized sudo usage by non-privileged user

**Data source:** `/var/log/auth.log` via Wazuh Agent

**Wazuh Rule:**
```xml
<rule id="5405" level="5">
  <if_sid>5401</if_sid>
  <match>user NOT in sudoers</match>
  <description>Unauthorized user attempted sudo</description>
  <mitre><id>T1548</id></mitre>
</rule>
```

**Threshold:** Any single occurrence

**MITRE:** T1548.004 (Abuse Elevation: Sudo)

**Test command:**
```bash
# As non-privileged user
sudo whoami
sudo passwd root
sudo visudo
```

**Expected alert level:** 5 (Warning)

---

## QRY-003: Cron Job Modification

**Use case:** Detect any modification to root crontab

**Data source:** `/var/log/syslog` via Wazuh Agent

**Wazuh Rule:**
```xml
<rule id="2833" level="8">
  <if_sid>2831</if_sid>
  <match>REPLACE</match>
  <description>Root's crontab entry changed</description>
  <mitre><id>T1053</id></mitre>
</rule>
```

**Threshold:** Any crontab modification

**MITRE:** T1053.006 (Scheduled Task: Cron)

**Test command:**
```bash
(crontab -l 2>/dev/null; echo "@reboot /tmp/malware.sh") | crontab -
```

**Expected alert level:** 8 (High)

---

## QRY-004: Scheduled Task Creation (Windows) [GAP - Fix Pending]

**Use case:** Detect scheduled task creation on Windows endpoints

**Data source:** Windows Security Event Log (Event ID 4698)

**Current status:** ❌ NOT COLLECTING — requires fix below

**Fix required:**
```xml
<!-- ossec.conf on Windows agent -->
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
  <query>Event/System[EventID=4698]</query>
</localfile>
```

**Wazuh Rule (after fix):**
```xml
<rule id="60001" level="7">
  <field name="win.system.eventID">^4698$</field>
  <description>Windows: Scheduled task created</description>
  <mitre><id>T1053.005</id></mitre>
</rule>
```

**MITRE:** T1053.005 (Scheduled Task/Job: Scheduled Task)

---

## QRY-005: File Staging in /tmp [GAP - Fix Pending]

**Use case:** Detect file creation/modification in /tmp (data staging indicator)

**Data source:** Wazuh syscheck (FIM)

**Current status:** ❌ NOT MONITORING — /tmp excluded from FIM

**Fix required:**
```xml
<!-- ossec.conf -->
<syscheck>
  <directories check_all="yes" realtime="yes">/tmp</directories>
  <directories check_all="yes" realtime="yes">/home</directories>
  <alert_new_files>yes</alert_new_files>
</syscheck>
```

**Wazuh Rule (after fix):**
```xml
<rule id="70001" level="8">
  <if_sid>550</if_sid>
  <match>\.tar\.gz$|\.zip$</match>
  <field name="file">/tmp/</field>
  <description>Archive created in /tmp — possible exfiltration staging</description>
  <mitre><id>T1560.001</id></mitre>
</rule>
```

**MITRE:** T1560.001 (Archive Collected Data)

---

## QRY-006: Hidden Directory Creation

**Use case:** Detect creation of dot-prefixed hidden directories

**Data source:** Wazuh syscheck (FIM)

**Wazuh Rule:**
```xml
<rule id="70002" level="7">
  <if_sid>554</if_sid>
  <match>/\.</match>
  <description>Hidden directory created — possible backdoor artifact</description>
  <mitre><id>T1564.001</id></mitre>
</rule>
```

**MITRE:** T1564.001 (Hide Artifacts: Hidden Files)

---

## QRY-007: PowerShell Hidden Execution (Windows)

**Use case:** Detect PowerShell with evasion flags

**Data source:** Windows Security/PowerShell event logs

**Wazuh Rule:**
```xml
<rule id="60002" level="8">
  <field name="win.eventdata.commandLine">-WindowStyle Hidden|-NonInteractive|-NoProfile</field>
  <description>PowerShell with evasion flags detected</description>
  <mitre><id>T1059.001</id></mitre>
</rule>
```

**MITRE:** T1059.001 (Command and Scripting Interpreter: PowerShell)

---

## Query Coverage Summary

| Query ID | Use Case | Status | MITRE |
|----------|----------|--------|-------|
| QRY-001 | SSH Brute Force | ✅ Validated | T1110.001 |
| QRY-002 | Sudo Escalation | ✅ Validated | T1548.004 |
| QRY-003 | Cron Modification | ✅ Validated | T1053.006 |
| QRY-004 | Win Scheduled Task | ❌ Gap (fix documented) | T1053.005 |
| QRY-005 | File Staging in /tmp | ❌ Gap (fix documented) | T1560.001 |
| QRY-006 | Hidden Directory | 🔄 Rule written, not tested | T1564.001 |
| QRY-007 | PowerShell Evasion | 🔄 Rule written, not tested | T1059.001 |
