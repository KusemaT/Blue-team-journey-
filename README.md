# Blue-team-journey-
Public portfolio from day one

# Day 01 — Wireshark: First Live Capture
**Date:** 7 March 2026  
**Tool:** Wireshark 4.4.)  
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
