# Network Traffic Analysis Report

**Analyst:** Michael Argaw  
**Date:** June 1, 2026  
**Environment:** VMware virtualized lab — Kali Linux (analyst station) 
+ Windows 11 (endpoint)  
**Tools:** tcpdump, Wireshark  
**Capture Files:** project_capture.pcap, icmp_capture.pcap, http_capture.pcap  

---

## Executive Summary

Network traffic was captured and analyzed between two virtual machines 
simulating a SOC analyst monitoring an enterprise endpoint. A total of 
51,137 packets were captured across three targeted pcap files. Analysis 
identified normal ICMP reconnaissance patterns, multiple TCP session 
establishments via the three-way handshake, and — critically — unencrypted 
HTTP traffic exposing Microsoft application download content in plaintext. 
Background Windows services were identified as the source of significant 
HTTP traffic volume, a finding relevant to SOC baseline profiling and 
anomaly detection.

---

## Environment

| Component | Details |
|-----------|---------|
| Analyst Machine | Kali Linux — 192.168.126.129 |
| Monitored Endpoint | Windows 11 — 192.168.126.128 |
| Network | VMware isolated host-only — 192.168.126.0/24 |
| Capture Interface | eth0 |
| Total Packets Captured | 51,137 |
| Capture Duration | ~60 seconds |

---

## Methodology

Traffic was captured using tcpdump with three targeted approaches:

1. **Full mixed capture** — all traffic on eth0 for baseline profiling
2. **ICMP-filtered capture** — BPF filter isolating ping traffic only
3. **HTTP-filtered capture** — BPF filter isolating TCP port 80 traffic

Captures were analyzed using both tcpdump command-line analysis and 
Wireshark for deep packet inspection, stream following, and statistical 
overview.

---

## Findings

### Finding 1 — Protocol Distribution

Analysis of the Protocol Hierarchy revealed the following traffic 
composition across 51,137 packets:

| Protocol | Packets | Percentage |
|----------|---------|------------|
| TCP | 40,779 | 79.7% |
| QUIC (encrypted) | 8,632 | 16.9% |
| HTTP (plaintext) | 1,050 | 2.1% |
| TLS (encrypted) | 3,956 | 7.7% |
| DNS | 1,038 | 2.0% |
| ICMP | 40 | 0.1% |
| ARP | 74 | 0.1% |

**Analyst Note:** The presence of both HTTP and QUIC/TLS traffic 
indicates the endpoint is making both encrypted and unencrypted 
connections simultaneously. The 1,050 unencrypted HTTP packets 
represent a measurable attack surface for on-path interception.

---

### Finding 2 — ICMP Reconnaissance Pattern

BPF-filtered capture isolated 40 ICMP packets representing 20 complete 
echo request/reply sequences between the two lab hosts.

**Key observations:**
- Source TTL of 64 (Kali Linux) vs reply TTL of 128 (Windows 11)
- TTL values are consistent with OS fingerprinting — Linux defaults 
  to TTL 64, Windows to TTL 128
- All sequences completed successfully — 0% packet loss
- Uniform 64-byte packet length — consistent with standard ping behavior

**SOC Relevance:** In a production environment, a high volume of ICMP 
requests from a single source to multiple hosts would indicate network 
reconnaissance or a ping sweep. TTL analysis can identify the OS of 
scanning hosts without additional tools.

---

### Finding 3 — TCP Three-Way Handshake

Wireshark analysis with filter `tcp.flags.syn == 1` revealed multiple 
TCP session establishments from the Windows 11 endpoint to external IPs.

**Handshake sequence confirmed:**
- **SYN** — `192.168.126.128` → `146.75.38.172:80` 
  Seq=0, Win=65535, MSS=1460
- **SYN-ACK** — `146.75.38.172` → `192.168.126.128` 
  Seq=0, Ack=1, Win=64240, MSS=1460
- **ACK** — `192.168.126.128` → `146.75.38.172` 
  Ack=1, Win=65535

**External IPs contacted by endpoint:**

| IP Address | Port | Sessions |
|------------|------|---------|
| 146.75.38.172 | 80 | Multiple — highest volume |
| 51.104.15.253 | 443 | Encrypted sessions |
| 40.79.141.152 | 443 | Encrypted sessions |
| 92.223.96.6 | 80 | Plaintext sessions |
| 23.54.127.170 | 80 | Plaintext sessions |

**SOC Relevance:** 81 distinct TCP conversations were observed from a 
single endpoint within 60 seconds. In a SOC environment, this volume 
and the presence of unrecognized external IPs would trigger a threat 
intelligence lookup against each destination.

---

### Finding 4 — Critical: Plaintext HTTP Payload Exposure

**This is the primary security finding of this analysis.**

Following a TCP stream on HTTP traffic to `146.75.38.172:80` revealed 
fully readable HTTP request and response headers including:

**Exposed Request Data:**
GET /filestreamingservice/files/d0c14d07-68b8-418c-a451-fbb06c9f3240 HTTP/1.1
Host: tlu.dl.delivery.mp.microsoft.com
User-Agent: Microsoft-Delivery-Optimization/10.1
Range: bytes=6291456-7340031

**Exposed Response Data:**
HTTP/1.1 206 Partial Content
Content-Disposition: filename=Microsoft.Xbox.TCUI_1.24.10001.0_neutral
Content-Type: application/octet-stream
Content-Length: 1048576
Date: Mon, 01 Jun 2026 18:36:10 GMT

**What this exposes:**
- Full file paths and filenames of software being downloaded
- Application version numbers and package identifiers  
- Download chunk ranges revealing total file sizes
- Timestamp and cache infrastructure details
- Over 1MB of binary application data per response chunk

**SOC Relevance:** Any on-path observer — a compromised router, a 
rogue access point, or an insider threat with network access — could 
intercept this traffic and identify exactly what software is being 
installed on this endpoint. In a corporate environment this constitutes 
an information disclosure vulnerability.

---

### Finding 5 — Background Service Traffic Identification

The HTTP traffic source was identified as Windows Delivery Optimization 
service — a built-in Windows component that downloads updates and app 
packages. This represents a SOC baseline profiling finding:

- Background services generate significant traffic independent of user 
  activity
- This traffic must be baselined to avoid false positive alerts
- The use of unencrypted HTTP by a Microsoft system service is a 
  vendor security concern worth documenting

---

## Tools & Commands Reference

| Command | Purpose |
|---------|---------|
| `sudo tcpdump -i eth0 -n -w capture.pcap` | Live full capture |
| `tcpdump -n -r capture.pcap 'icmp'` | Filter ICMP traffic |
| `tcpdump -n -r capture.pcap 'tcp port 80'` | Filter HTTP traffic |
| `tcpdump -n -r capture.pcap -X -c 10` | Hex/ASCII payload inspection |
| `tcpdump -n -r capture.pcap 'host A and host B'` | Filter by conversation |
| Wireshark: `icmp` | Display filter — ICMP only |
| Wireshark: `tcp.flags.syn == 1` | Display filter — SYN packets |
| Wireshark: `http` | Display filter — HTTP only |
| Wireshark: Follow → TCP Stream | Raw stream reconstruction |
| Statistics → Protocol Hierarchy | Traffic composition overview |
| Statistics → Conversations → TCP | Session enumeration |

---

## Recommendations

**R1 — Enforce HTTPS for all software delivery**
Microsoft Delivery Optimization should be configured to use HTTPS 
exclusively. Group Policy: Computer Configuration → Administrative 
Templates → Windows Components → Delivery Optimization → set 
Download Mode to require encrypted transfers.

**R2 — Implement network monitoring baselines**
Background service traffic must be baselined in SIEM rules to 
distinguish legitimate Windows Update activity from malicious 
beaconing to similar CDN infrastructure. Key baseline metrics: 
destination IP ranges, transfer volumes, and timing patterns.

**R3 — Deploy threat intelligence enrichment on external IPs**
All 81 external IPs identified in the conversation table should be 
automatically enriched against threat intelligence feeds (VirusTotal, 
Shodan, AbuseIPDB) in a production SOC environment.

**R4 — Monitor for ICMP-based reconnaissance**
Implement SIEM alerts for ICMP volume thresholds exceeding 10 
requests/second from a single source — a common indicator of 
automated ping sweep tools.

---

## Conclusion

This analysis demonstrates core SOC analyst capabilities: targeted 
packet capture using BPF filters, protocol-level traffic analysis, 
TCP session reconstruction, and plaintext payload inspection. The 
primary finding — unencrypted HTTP exposing application download 
content — represents a real information disclosure risk present in 
default Windows configurations. All findings were documented with 
supporting packet-level evidence suitable for escalation to a 
security engineering team.

---

## Screenshots

*See /screenshots directory in this repository*

- `protocol_hierarchy.png` — Protocol distribution across 51,137 packets
- `icmp_filter.png` — ICMP echo request/reply sequence
- `tcp_handshake.png` — Three-way handshake SYN/SYN-ACK/ACK
- `http_stream.png` — HTTP filter showing active sessions
- `http_plaintext.png` — TCP stream showing exposed plaintext content
- `conversations_tcp.png` — 81 TCP conversations from single endpoint
