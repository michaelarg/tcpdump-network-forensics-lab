cat > ~/Documents/Michael_projects/tcpdump-network-forensics-lab/reports/investigation-report.md << 'EOF'
# Network Forensics Investigation Report

---

## Case Information

| Field | Details |
|-------|---------|
| **Case Title** | DNS Traffic Analysis — ESPN Web Browsing Investigation |
| **Analyst** | Michael Argaw |
| **Date** | June 1, 2026 |
| **Environment** | Kali Linux |
| **Tools Used** | tcpdump, awk, wc |
| **Evidence File** | dns.pcap |
| **Source** | Chris Sanders Public Packet Repository |

---

## Executive Summary

A packet capture file (dns.pcap) containing 7 DNS packets was analyzed
using tcpdump on Kali Linux. The investigation identified a single client
host (172.16.16.154) resolving multiple ESPN and content delivery network
(CDN) domains through a public DNS server (4.2.2.1). All traffic occurred
within less than one second, consistent with a user loading the ESPN
website in a browser. No malicious activity, suspicious domains, or
anomalous behavior was detected.

---

## Objectives

- Identify all hosts communicating in the capture
- Determine what protocols and ports were in use
- Identify what domains were being resolved
- Assess whether any traffic was suspicious or malicious
- Document all findings in a structured analyst report

---

## Evidence Analysis

### 1. Capture Overview

```bash
tcpdump -n -r dns.pcap | wc -l
```

| Field | Value |
|-------|-------|
| Total Packets | 7 |
| Capture Duration | Less than 1 second |
| Timestamp Range | 15:59:15.346584 — 15:59:15.983886 |
| Link Type | EN10MB (Ethernet) |
| Snapshot Length | 262,144 bytes |

---

### 2. Host Identification

```bash
tcpdump -n -r dns.pcap | awk '{print $3}' | cut -d'.' -f1-4 | sort | uniq -c | sort -rn
```

| IP Address | Role | Description | Packet Count |
|------------|------|-------------|--------------|
| 4.2.2.1 | DNS Server | Level 3 Communications Public DNS | 7 |
| 172.16.16.154 | Client Host | Internal host making DNS queries | 7 |

**Analysis:** Two hosts were involved in this capture. The client
172.16.16.154 sent DNS queries and received responses from the public
DNS server 4.2.2.1. The private IP range 172.16.0.0/12 confirms the
client is an internal network host.

---

### 3. Protocol Analysis

```bash
tcpdump -n -r dns.pcap 'udp port 53' -#
```

| Protocol | Port | Count | Description |
|----------|------|-------|-------------|
| UDP | 53 | 7 | DNS — Domain Name System |

**Analysis:** All 7 packets were DNS (UDP port 53). The BPF filter
confirmed no other protocols were present in this capture. UDP is the
standard transport protocol for DNS queries and responses under 512 bytes.

---

### 4. DNS Query Analysis

```bash
tcpdump -n -r dns.pcap -v
```

| Packet | Timestamp | Domain Queried | Resolved IP | CDN/Provider |
|--------|-----------|----------------|-------------|--------------|
| 1 | 15:59:15.346584 | www.espn.com | 68.71.212.158 | ESPN Direct |
| 2 | 15:59:15.558135 | espn.go.com | 199.181.133.61 | ESPN/Disney |
| 3 | 15:59:15.846451 | cdn.optimizely.com | 72.21.91.8 | Edgecast CDN |
| 4 | 15:59:15.847604 | a1.espncdn.com | 72.246.56.35, 72.246.56.83 | Akamai CDN |
| 5 | 15:59:15.898379 | assets.espn.go.com | 69.31.75.194, 69.31.75.203 | Akamai CDN |
| 6 | 15:59:15.979495 | a2.espncdn.com | 72.246.56.83, 72.246.56.35 | Akamai CDN |
| 7 | 15:59:15.983886 | a4.espncdn.com | 72.246.56.35, 72.246.56.83 | Akamai CDN |

**Analysis:** The client resolved 7 domains all related to ESPN and its
content delivery infrastructure. The presence of Akamai and Edgecast CDN
domains is normal — major websites use CDNs to serve static assets like
images, scripts, and stylesheets faster. The domain cdn.optimizely.com
is an A/B testing and analytics platform commonly embedded in websites.

---

### 5. Packet Structure Analysis

```bash
tcpdump -n -r dns.pcap -v
```

| Field | Value | Significance |
|-------|-------|--------------|
| TTL | 54 | Consistent across all packets — same DNS server |
| Flags | [DF] Don't Fragment | Normal DNS behavior |
| Protocol | UDP (17) | Standard DNS transport |
| Packet Size | 68–171 bytes | Normal DNS response sizes |
| CNAME Records | Multiple | Normal CDN redirection chain |

**Analysis:** The TTL of 54 on all packets indicates they all originated
from the same DNS server at approximately the same network distance.
The [DF] Don't Fragment flag is standard for DNS UDP packets. Multiple
CNAME records are expected when resolving CDN-hosted content as they
represent the redirection chain from the original domain to the CDN edge node.

---

### 6. Timeline Reconstruction
