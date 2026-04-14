# Blue-team-journey-
Public portfolio from day one

# Day 01 — Wireshark: First Live Capture
**Date:** 7 March 2026  
**Tool:** Wireshark 4.4.  
**Category:** Network Analysis  
**ATT&CK Technique:** T1040 — Network Sniffing  


## Overview
First hands-on session using Wireshark to capture and analyse live network traffic from my own machine. Goal was to understand what normal traffic looks like before learning to spot anomalies.

---

## Setup
- **OS:** [Macbook Pro - Intel core i7]
- **Interface captured:** Wi-Fi / Ethernet
- **Capture duration:** ~2 minutes
- **Activity during capture:** Browsed to a website to generate traffic

---

## What I Found

### DNS Traffic (`dns` filter)
DNS queries resolve domain names to IP addresses. Every time a browser loads a page, it fires DNS requests first.

| Domain Queried | Query Type | Notes |
|---|---|---|
| apple.com | A | Normal browsing |
| fbcdn.net | A | CDN asset load |
| analytics.google.com | A | Third-party tracker |

**Observation:** A single page load triggered multiple DNS queries — not just the main domain, but subdomains and third-party services I didn't explicitly navigate to.

---

### HTTP Traffic (`http` filter)
HTTP requests show the raw communication between browser and server before encryption (HTTPS hides this).

**Sample GET request found:**

**Key fields noted:**
- **Method:** GET — requesting a resource
- **Host:** the destination server
- **User-Agent:** reveals OS and browser — analysts use this to fingerprint devices and spot anomalies (e.g. a User-Agent string for curl or Python requests on a user workstation is suspicious)

---

## Key Takeaways
1. A single page visit generates far more network traffic than expected — DNS, HTTP, CDN requests, third-party trackers
2. DNS is a goldmine for SOC analysts — malware uses DNS for C2 beaconing (T1071.004) and data exfiltration
3. HTTP User-Agent strings can identify non-human traffic — a PowerShell or Python User-Agent on an endpoint is an immediate red flag
4. Wireshark filters (`dns`, `http`, `ip.addr == x.x.x.x`) are essential for cutting through noise

---

## SOC Relevance
> In a real SOC investigation, an analyst would look for:
> - DNS queries to unusual/newly-registered domains (potential C2)
> - HTTP traffic over port 443 (HTTPS tunnelling)
> - Abnormally large DNS responses (DNS exfiltration)
> - User-Agent strings that don't match the expected browser on that asset

---

## MITRE ATT&CK Mapping
| Technique | ID | Relevance |
|---|---|---|
| Network Sniffing | T1040 | What Wireshark does — also how attackers capture credentials on a network |
| Application Layer Protocol: Web Protocols | T1071.001 | HTTP/HTTPS used for C2 |
| Application Layer Protocol: DNS | T1071.004 | DNS used for C2 beaconing |

---

## Tools Used
- Wireshark — packet capture and analysis
- Display filters: `dns`, `http`, `ip.addr`, `Follow HTTP Stream`

---

## Next Steps
- [ ] Analyse a PCAP from CyberDefenders to see malicious traffic patterns
- [ ] Learn `tcp.flags.syn == 1` filter for port scan detection
- [ ] Compare normal DNS to DNS beaconing behaviour

## Week 1 Plan — 7 March 2026

This week I'm setting up my home lab environment (VirtualBox, Kali Linux, Windows 10 VM) and completing my first live Wireshark capture, with findings documented in this repo. My main focus is beginning the CompTIA Security+ SY0-701 course via Professor Messer, starting the TryHackMe Pre-Security path, and building my Anki deck to 30+ cards covering OSI model, CIA triad, and security control types. One thing I'm unsure about: how Sysmon event logs will look once ingested into Splunk — I'll find out by end of week.
---

## Components

# Home Lab Setup — Blue Team SOC Environment
**Date:** 8 April 2026
**Category:** Infrastructure
**Status:** Live and operational

---

## Overview
Built a home SOC lab environment to practice blue team skills outside of theoretical study. The lab simulates a small enterprise environment with an attacker machine, a target Windows endpoint, and a SIEM for log analysis and detection rule development.

---

## Lab Architecture
### Hypervisor
- **Tool:** VirtualBox (free)
- **Purpose:** Runs all virtual machines in isolation from the host

### Attacker Machine — Kali Linux
- **Image:** Pre-built VirtualBox OVA from kali.org
- **Purpose:** Simulating attacker techniques for detection testing
- **Key tools:** Nmap, Wireshark, Impacket, Hydra, BloodHound

### Target Machine — Windows 10
- **Image:** Microsoft Evaluation ISO (90-day free licence)
- **Spec:** 2 CPU cores, 4GB RAM, 50GB disk
- **Purpose:** Simulates a corporate endpoint — the thing being attacked and monitored

### Endpoint Telemetry — Sysmon
- **Config:** SwiftOnSecurity sysmon-config (community-maintained)
- **Purpose:** Generates rich Windows event logs far beyond default Windows logging
- **Key Event IDs monitored:**

| Event ID | Description |
|----------|-------------|
| 1 | Process creation (includes full command line) |
| 3 | Network connection (destination IP and port) |
| 8 | CreateRemoteThread (process injection indicator) |
| 10 | ProcessAccess (LSASS access — credential dumping indicator) |
| 11 | File creation |
| 22 | DNS query |

### SIEM — Splunk Enterprise
- **Licence:** Free trial (500MB/day ingest, no time limit)
- **Access:** http://localhost:8000
- **Data source:** Sysmon logs via Windows Event Log input
- **Purpose:** Central log aggregation, detection rule testing, alert creation

---

## Detection Rules Built

### 1. Brute Force Detection
```spl
index=* EventCode=4625
| bin _time span=10m
| stats count by Account_Name, _time
| where count >= 5
```
**ATT&CK:** T1110 — Brute Force
**Trigger:** 5+ failed logons for same account within 10 minutes

### 2. Suspicious PowerShell Execution
```spl
index=* EventCode=1 Image=*powershell*
CommandLine="*-ExecutionPolicy Bypass*"
| table _time, CommandLine, ParentImage
```
**ATT&CK:** T1059.001 — PowerShell
**Trigger:** Any PowerShell execution with bypass flags

### 3. AD Enumeration Detection
```spl
index=* EventCode=4688
(CommandLine="*net * /domain*" OR CommandLine="*net user*")
| table _time, Account_Name, CommandLine
```
**ATT&CK:** T1087 — Account Discovery
**Trigger:** net.exe enumeration commands

---

## What This Enables
- Testing detection rules against real attack simulations
- Practising Splunk SPL queries against actual log data
- Simulating ATT&CK techniques and verifying detections fire
- Building a portfolio of documented detection engineering work

---

## Next Steps
- [ ] Add Active Directory domain controller (lab.local)
- [ ] Ingest Zeek network logs from Security Onion
- [ ] Expand detection rule library to 10+ rules
- [ ] Add YARA rules for malware detection
- [ ] Simulate Kerberoasting and build detection
