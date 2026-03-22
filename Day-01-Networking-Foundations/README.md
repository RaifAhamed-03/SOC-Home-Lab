# Day 1 – Networking Foundations for SOC Analysts

## Objective
Build a solid foundation in **network traffic analysis** using **Wireshark** focusing on identifying normal network behavior versus potential security risks from a SOC analyst perspective.

## Key Learning Outcomes
- Alert triage
- Incident investigation
- Threat detection
- Network-based attack identification

## Tools Used
- **Wireshark** – Network packet capture & analysis
- **Windows Command Line / PowerShell** – Traffic generation
- **Public DNS (8.8.8.8)** – Connectivity testing

---

## Tasks Performed

### 1. Live Packet Capture
- Captured real-time network traffic on the active interface.
- Generated traffic by browsing websites and sending ICMP requests.
- Verified packet flow and correct interface selection.

**Commands Used:** Command Line  
`ping -n 20 8.8.8.8`

![Command Line Ping Traffic](screenshots/Ping.png)  

![Live Packet Capture](screenshots/day1_live_capture.png)

### 2. Protocol Analysis
Applied Wireshark filters to analyze traffic commonly monitored in SOC environments.

| Protocol | Filter Used | SOC Relevance |
| :--- | :--- | :--- |
| DNS | `dns` | Detects command-and-control, data exfiltration |
| ICMP | `icmp` | Detects reconnaissance and DoS attacks |
| TCP | `tcp` | Session establishment, brute force attempts |
| HTTPS | `tcp.port == 443` | Encrypted communication monitoring |

**DNS Traffic:**
![DNS Traffic Filter](screenshots/day1_dns_filter.png)

**ICMP Traffic:**
![ICMP Traffic Filter](screenshots/day1_icmp_filter.png)

**TCP Handshake:**
![TCP Traffic Filter](screenshots/day1_tcp_handshake.png)

### 3. TCP Three-Way Handshake Analysis
- Identified `SYN → SYN-ACK → ACK` sequence.
- Determined:
  - Initiating client
  - Destination server
  - Service ports
- Noted abnormal/incomplete handshakes may indicate:
  - Port scanning
  - SYN flood attacks

**Wireshark Filter Command:** `tcp.flags.syn==1 && tcp.flags.ack==0`

![TCP Handshake Filter](screenshots/day1_tcp_handshake.png)

### 4. Normal vs Suspicious Traffic Evaluation

**Baseline normal traffic:**
- Standard DNS query patterns
- Normal ICMP request frequency
- Completed TCP sessions

**Suspicious activity to monitor:**
- Repeated DNS queries to same domain
- High-frequency ICMP traffic
- Multiple SYN packets without completing handshake
- Connections to unusual or unknown external IPs

![Suspicious Traffic Evaluation](screenshots/day1_suspicious_traffic.png)

---

## Security Concepts Demonstrated
- Network reconnaissance detection
- Protocol-level traffic analysis
- Baseline behavior identification
- Early-stage attack awareness
- SOC triage mindset

## Key Observations
- DNS traffic is a valuable indicator for malware or C2 detection.
- ICMP traffic can be abused for scanning and flooding.
- TCP handshake analysis helps detect abnormal connection attempts.
- Establishing normal network patterns is essential before writing or tuning SIEM rules.

## Skills Gained
- Packet capture & filtering
- Network traffic interpretation
- SOC-style traffic analysis
- Threat detection mindset
- Documentation & reporting
