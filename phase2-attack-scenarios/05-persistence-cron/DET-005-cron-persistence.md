# Detection Spec: Cron Job Persistence

**Spec ID:** DET-005  
**MITRE Technique:** T1053.006  
**Status:** ✅ VALIDATED  
**Last Tested:** 2026-06-27  

---

## Telemetry Source

```
Host:        kali (192.168.100.4)
Log source:  /var/log/syslog
Agent:       Wazuh Agent 004
Process:     cron daemon
```

---

## Detection Condition

```
IF: root crontab is modified (REPLACE or ADD)
FROM: any user
AT: any time
THEN: trigger crontab modification alert
```

---

## Wazuh Rule

```xml
<!-- Rule 2833: Root crontab modified -->
<rule id="2833" level="8">
  <if_sid>2831</if_sid>
  <match>REPLACE</match>
  <description>Root's crontab entry changed.</description>
  <mitre>
    <id>T1053</id>
  </mitre>
</rule>
```

---

## Attack Executed

```bash
# Step 1: Create malicious backdoor script
cat > /tmp/update_check.sh << 'EOF'
#!/bin/bash
# Masquerading as system update
mkdir -p /var/lib/.hidden
echo "$(date): Persistence check executed" >> /var/lib/.hidden/update.log
EOF
chmod +x /tmp/update_check.sh

# Step 2: Install persistence via crontab
(crontab -l 2>/dev/null; echo "@reboot /tmp/update_check.sh") | crontab -
(crontab -l 2>/dev/null; echo "*/5 * * * * /tmp/update_check.sh") | crontab -

# Verify
crontab -l
# Output:
# @reboot /tmp/update_check.sh
# */5 * * * * /tmp/update_check.sh
```

---

## Raw Syslog Output

```
Jun 27 01:52:09 kali crontab[66995]: (root) REPLACE (root)
Jun 27 01:52:30 kali crontab[67213]: (root) REPLACE (root)
```

---

## Wazuh Alert Output

```
** Alert 1782525129.2650447: - syslog,cron,pci_dss_10.2.7,gpg13_4.13,
   gdpr_IV_35.7.d,gdpr_IV_32.2,hipaa_164.312.b,nist_800_53_AU.14,
   nist_800_53_AU.6,nist_800_53_AC.6,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3

Rule: 2833 (level 8) → 'Root's crontab entry changed.'
Jun 27 01:52:09 kali crontab[66995]: (root) REPLACE (root)

** Alert 1782525152.2650831: - syslog,cron,pci_dss_10.2.7,gpg13_4.13,
   gdpr_IV_35.7.d,gdpr_IV_32.2,hipaa_164.312.b,nist_800_53_AU.14,
   nist_800_53_AU.6,nist_800_53_AC.6,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3

Rule: 2833 (level 8) → 'Root's crontab entry changed.'
Jun 27 01:52:30 kali crontab[67213]: (root) REPLACE (root)
```

---

## Detection Outcome

| Metric | Result |
|--------|--------|
| Detected | ✅ YES |
| Alert rule fired | Rule 2833 (Level 8) |
| Detection latency | <5 seconds |
| Both cron jobs detected | ✅ YES (2 separate alerts) |
| False positives | None observed |

---

## Compliance Tags

```
PCI-DSS:  10.2.7 (audit log monitoring)
GDPR:     IV.35.7.d (security monitoring)
HIPAA:    164.312.b (audit controls)
NIST:     AU-14 (session audit)
NIST:     AC-6 (least privilege)
```

---

## Failure Cases / Edge Cases

| Scenario | Result | Notes |
|----------|--------|-------|
| System-level cron (/etc/cron.d/) | ❌ NOT tested | Requires separate FIM rule |
| Cron via at command | ❌ NOT tested | Different log source |
| Cron via systemd timer | ❌ NOT detected | Different mechanism entirely |
| Backdoor execution after reboot | ❌ NOT tested | Would need reboot simulation |

---

## Backdoor Artifacts

```
Script location:    /tmp/update_check.sh
Hidden directory:   /var/lib/.hidden/
Log file:           /var/lib/.hidden/update.log
Cron triggers:      @reboot, */5 * * * *
Masquerade name:    "update_check" (looks like system tool)
```

---

## Remediation

```bash
# Remove persistence
crontab -r                                    # Remove all cron jobs
rm -f /tmp/update_check.sh                   # Remove backdoor script
rm -rf /var/lib/.hidden/                     # Remove hidden directory

# Harden cron access
echo "root" > /etc/cron.allow               # Only root can use cron
chmod 600 /etc/cron.d/*                     # Restrict cron.d files

# Restart cron daemon
sudo systemctl restart cron
```

---

## Evidence Files

- `screenshots/05-backdoor-script-created.png` — Script creation
- `screenshots/06-cron-jobs-added.png` — Both cron jobs verified
- `screenshots/07-wazuh-cron-detection.png` — Rule 2833 firing
