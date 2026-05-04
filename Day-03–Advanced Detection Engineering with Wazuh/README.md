# Day 3 – Advanced Detection Engineering with Wazuh

## Objective
Develop advanced detection engineering skills by creating custom rules, integrating threat intelligence, and implementing automated active response for malicious file detection using Wazuh SIEM.

## Key Learning Outcomes
| # | Learning Outcome |
|---|------------------|
| 1 | Create custom detection rules for suspicious processes |
| 2 | Integrate VirusTotal threat intelligence for malware detection |
| 3 | Configure File Integrity Monitoring (FIM) for critical directories |
| 4 | Implement active response for automated file quarantine |
| 5 | Map detections to MITRE ATT&CK framework |

## Tools Used
| Tool | Purpose |
|------|---------|
| **Wazuh SIEM** | Detection and response engine |
| **VirusTotal API** | Cloud-based threat intelligence (70+ antivirus engines) |
| **File Integrity Monitoring (FIM)** | Real-time file change detection |
| **Active Response** | Automated quarantine and remediation |
| **Custom Rules (XML)** | Detection logic engineering |

---

## Tasks Performed

### 1. File Integrity Monitoring (FIM) Configuration

Configured real-time file integrity monitoring on Ubuntu Client to detect file creations, modifications, and deletions in critical directories.

| Directory | Monitoring Type | Purpose |
|-----------|-----------------|---------|
| `/home/socuser/Downloads` | Real-time | Detect user-downloaded malware |
| `/tmp` | Real-time | Detect temporary malicious files |

**FIM Configuration (`ossec.conf`):**
```xml
<syscheck>
    <directories realtime="yes" check_all="yes">/home/socuser/Downloads</directories>
    <directories realtime="yes" check_all="yes">/tmp</directories>
</syscheck>
```

**FIM Configuration:**  
![FIM Configuration](screenshots/01_fim_configuration.png)  
*File Integrity Monitoring configured for Downloads and /tmp directories with real-time scanning enabled.*

---

### 2. VirusTotal Threat Intelligence Integration

Integrated VirusTotal API to automatically scan newly created files against 70+ antivirus engines for malware detection.

**Step 1: Obtain VirusTotal API Key**
- Created free VirusTotal account
- Generated API key for integration

**Step 2: Configure Integration (ossec.conf)**
```xml
<integration>
    <name>virustotal</name>
    <api_key>YOUR_API_KEY</api_key>
    <group>syscheck</group>
    <alert_format>json</alert_format>
</integration>
```

**Step 3: VirusTotal Alert Details**
| Field | Value |
|-------|-------|
| **Rule ID** | 87105 |
| **Level** | 12 (High) |
| **Description** | VirusTotal: Alert - 60+ engines detected malicious file |
| **MITRE Technique** | T1203 – Exploitation for Client Execution |

**VirusTotal Alert in Wazuh Dashboard:**  
![VirusTotal Alert](screenshots/02_virustotal_alert.png)  
*VirusTotal integration successfully detected EICAR test file with 60+ antivirus engines flagging it as malicious. Alert shows file path, hash values, and permalink to full VirusTotal report.*

**Alert JSON Output:**
```json
{
  "rule.id": 87105,
  "rule.level": 12,
  "data.virustotal.positives": 60,
  "data.virustotal.total": 66,
  "data.virustotal.source.file": "/tmp/eicar.com",
  "data.virustotal.permalink": "https://www.virustotal.com/gui/file/..."
}
```

---

### 3. Active Response Implementation

Created an active response script to automatically quarantine and remove malicious files detected by VirusTotal.

**Active Response Script (remove-threat.sh):**
```bash
#!/bin/bash

LOG_FILE="/var/ossec/logs/active-responses.log"

ACTION=$1
USER=$2
IP=$3
ALERT_ID=$4
RULE_ID=$5
AGENT_ID=$6
FILE_PATH=$7

# Remove the malicious file
if [ -n "$FILE_PATH" ] && [ -f "$FILE_PATH" ]; then
    echo "$(date): Removing malicious file: $FILE_PATH" >> ${LOG_FILE}
    rm -f "$FILE_PATH"
    echo "$(date): File removed successfully" >> ${LOG_FILE}
fi

# Additional cleanup for EICAR files
find /tmp /home -name "*eicar*" -type f 2>/dev/null | while read file; do
    echo "$(date): Found and removing: $file" >> ${LOG_FILE}
    rm -f "$file"
done
```

**Active Response Configuration (ossec.conf):**
```xml
<command>
    <name>remove-threat</name>
    <executable>remove-threat.sh</executable>
    <timeout_allowed>no</timeout_allowed>
</command>

<active-response>
    <disabled>no</disabled>
    <command>remove-threat</command>
    <location>local</location>
    <rules_id>87105</rules_id>
    <fields>
        <field>file</field>
    </fields>
</active-response>
```

**Active Response Logs:**  
![Active Response Logs](screenshots/03_active_response_logs.png)  
*Active response script successfully triggered, finding and removing malicious EICAR test files from `/tmp` directory.*

---

### 4. End-to-End Malware Detection Test

Simulated a real-world malware detection scenario using the EICAR test file (safe, industry-standard test file).

**Test Execution Command:**
```bash
# Create EICAR test file (simulates malware download)
cat > /tmp/eicar.com << 'EOF'
X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
EOF
```

**Detection Timeline:**
| Time | Event |
|------|-------|
| 0 sec | EICAR file created in `/tmp/` directory |
| 5 sec | FIM detects file creation and triggers VirusTotal scan |
| 10 sec | VirusTotal returns detection (60+ engines) |
| 15 sec | Rule 87105 generates high-severity alert |
| 20 sec | Active response script executes |
| 25 sec | Malicious file automatically removed |

**Malware Detection Alert:**  
![Malware Detection](screenshots/04_malware_detection.png)  
*Complete end-to-end detection showing file creation, VirusTotal alert, MITRE ATT&CK mapping, and automated response.*

---

### 5. MITRE ATT&CK Framework Mapping

| Detected Activity | MITRE Technique ID | Tactic |
|-------------------|---------------------|--------|
| Malicious file download | T1203 – Exploitation for Client Execution | Execution |
| File executed from /tmp | T1059 – Command and Scripting Interpreter | Execution |
| Malware detection | T1204 – User Execution | Execution |
| Automated remediation | T1562 – Impair Defenses | Defense Evasion |

---

## Normal vs Suspicious Activity Evaluation

**Activity Type: File Creation in /tmp**  
**Normal Baseline:** Temporary application files  
**Suspicious Indicator:** Executable files, EICAR signature

**Activity Type: Downloaded Files**  
**Normal Baseline:** Known software installers  
**Suspicious Indicator:** Unknown hash, high detection rate

**Activity Type: Antivirus Detection**  
**Normal Baseline:** 0 detections  
**Suspicious Indicator:** 60+ engines positive

---

## Observed Suspicious Patterns

- EICAR test file detected by 60+ antivirus engines (Malware indicator)
- File created in `/tmp` directory (Common malware location)
- Automated removal triggered within 25 seconds (Successful response)
- MITRE ATT&CK mapping confirmed (T1203, T1059, T1562)

---

## MITRE ATT&CK Framework Mapping

**Detected Activity: Malicious file download**  
**MITRE Technique ID:** T1203 – Exploitation for Client Execution  
**Tactic:** Execution

**Detected Activity: File executed from /tmp**  
**MITRE Technique ID:** T1059 – Command and Scripting Interpreter  
**Tactic:** Execution

**Detected Activity: Malware detection**  
**MITRE Technique ID:** T1204 – User Execution  
**Tactic:** Execution

**Detected Activity: Automated remediation**  
**MITRE Technique ID:** T1562 – Impair Defenses  
**Tactic:** Defense Evasion

---

## Security Concepts Demonstrated

- Threat intelligence integration (VirusTotal API)
- File Integrity Monitoring (FIM) for compliance (PCI-DSS 11.5)
- Rule-based detection (Custom + Built-in rules)
- Active response and automated remediation
- MITRE ATT&CK framework mapping
- End-to-end SOC detection workflow

---

## Key Observations

- VirusTotal integration provides immediate access to 70+ antivirus engines without local resource overhead.
- File Integrity Monitoring is essential for detecting malware downloads in real-time.
- Active response reduces SOC analyst workload by automating ransomware/malware response.
- MITRE ATT&CK mapping provides context for alert prioritization.
- The EICAR test file is an invaluable tool for safe testing of security controls.

---

## SOC Use Case: Real-World Application

In a real SOC environment, this analysis enables:

**Use Case: Ransomware Detection**  
**Detection Method:** Mass file changes (FIM alerts)  
**Response Action:** Isolate endpoint, block C2 communication

**Use Case: Malware Download**  
**Detection Method:** VirusTotal hash lookup  
**Response Action:** Quarantine file, alert SOC

**Use Case: Suspicious File**  
**Detection Method:** Unknown hash, low reputation  
**Response Action:** Block execution, sandbox analysis

**Use Case: Persistence Mechanism**  
**Detection Method:** File creation in startup folders  
**Response Action:** Automated removal, investigation

---

## Skills Gained

- Threat intelligence API integration
- File Integrity Monitoring configuration
- Active response script development
- Custom detection rule creation
- MITRE ATT&CK framework application
- End-to-end SOC workflow implementation

---

## Wazuh Configuration Files Reference

| File | Purpose | Location |
|------|---------|----------|
| `ossec.conf` | Main configuration | `/var/ossec/etc/` |
| `local_rules.xml` | Custom detection rules | `/var/ossec/etc/rules/` |
| `remove-threat.sh` | Active response script | `/var/ossec/active-response/bin/` |
| `active-responses.log` | Response execution log | `/var/ossec/logs/` |
| `integrations.log` | Threat intel integration log | `/var/ossec/logs/` |

---

## Dashboard Screenshot Summary

| # | Screenshot | Purpose |
|---|------------|---------|
| 1 | `01_fim_configuration.png` | FIM configuration for critical directories |
| 2 | `02_virustotal_alert.png` | VirusTotal detection alert (Rule 87105) |
| 3 | `03_active_response_logs.png` | Active response execution logs |
| 4 | `04_malware_detection.png` | Complete end-to-end detection |

---

## Alert Summary (Rule 87105 - VirusTotal Detections)

| Timestamp | File Path | Positives | Total | Detection Rate |
|-----------|-----------|-----------|-------|----------------|
| 2026-05-04 08:21:41 | `/tmp/eicar.com` | 65 | 67 | 97% |
| 2026-05-04 08:37:33 | `/tmp/ar-test-eicar.com` | 64 | 66 | 97% |
| 2026-05-04 08:37:38 | `/tmp/eicar.com` | 64 | 66 | 97% |
| 2026-05-04 08:40:17 | `/tmp/eicar.com` | 60 | 66 | 91% |

---

## Lab Architecture Summary

```
Internet
    │
    ▼
┌─────────────────────────────────────────────────────────────┐
│                    Wazuh Manager (192.168.3.102)            │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │ VirusTotal API  │  │ Active Response │                   │
│  │ Integration     │  │ Script          │                   │
│  └─────────────────┘  └─────────────────┘                   │
│           │                    │                            │
│           ▼                    ▼                            │
│  ┌─────────────────────────────────────────┐                │
│  │         Wazuh Rules (87105)             │                │
│  │  File Creation → VT Scan → Alert        │                │
│  └─────────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ Port 1514 (Agent Communication)
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              Ubuntu Client (192.168.2.10)                   │
│  ┌─────────────────┐  ┌─────────────────┐                   │
│  │   FIM Monitor   │  │ Wazuh Agent     │                   │
│  │  /tmp, Downloads│  │                 │                   │
│  └─────────────────┘  └─────────────────┘                   │
│           │                    │                            │
│           ▼                    ▼                            │
│  ┌─────────────────────────────────────────┐                │
│  │      Malicious File (EICAR)             │                │
│  │         → Detected → Removed             │               │
│  └─────────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

---
