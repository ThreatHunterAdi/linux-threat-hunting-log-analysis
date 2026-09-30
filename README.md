# Project: Linux Threat Hunting & Raw Packet Log Analysis

## Executive Summary
This project demonstrates a manual incident triage of an enterprise network intrusion targeting a Linux deployment. Due to deliberate asset-constrained environments mimicking zero-trust infrastructure, analysis was performed directly on raw session payloads to dissect a live Redtail Bash Script compromise vector without relying on automated parsing layers.

## Threat Analysis Metrics
* **Victim Hostname / OS:** Ubuntu-Server-22-04 (Identified via command-line agent string `curl/7.88.1`)
* **Victim IP Address:** 172.16.4.191
* **Victim MAC Address:** 00:15:5d:04:19:61
* **Malicious URL Visited:** `http://45.202.35[.]190/sh`
* **Attacker C2 Infrastructure IP:** 45.202.35.190

---

## Technical Walkthrough & Evidence

### 1. Delivery Vector & Command-and-Control Isolation
By analyzing the raw unencrypted inbound session tracking, I captured a direct external callback targeting the infrastructure. The compromised host initiated an outbound HTTP request to grab a remote execution payload.
![Inbound C2 Connection](screenshot1.png)

*Figure 1: Raw connection capture exposing the remote host header and target resource endpoint.*

### 2. Malicious Payload Dissection
The downloaded payload is an automated deployment bash script configured to run secondary execution routines out of volatile directory storage (`/tmp`). 
![Malicious Script Analysis](screenshot2.png)

*Figure 2: Exploded view of the script logic showing redundant curl fallback routines and remote descriptor mapping.*

### 3. Incident Triage Environment Setup
To ensure forensic isolation and prevent data cross-contamination on endpoint storage channels, raw files were structured across dedicated secondary storage targets within a sandboxed virtual container environment.
![Forensic Workspace Environment](screenshot3.png)

*Figure 3: Dedicated sandbox architecture setup used during log extraction.*

---

## Prescriptive Containment & Remediation Playbook
1. **Network Layer Isolation:** Deploy an immediate egress block on the perimeter firewall for IP `45.202.35.190`.
2. **Endpoint Triage:** Terminate all active processes spawning out of the `/tmp` directory on the target Linux machine. Run an advanced rootkit checking scan (e.g., `chkrootkit`) to verify system binaries have not been altered.
3. **Log Audit:** Review local authentication logs (`/var/log/auth.log`) to deduce how the attacker originally authenticated (e.g., SSH brute force) to trigger the initial `curl` download execution sequence.
