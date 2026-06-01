# TCPdump Commands Log
## Project: DNS Pcap Investigation
## Analyst: Michael Argaw
## Date: June 1, 2026
## Environment: Kali Linux

---

## Command 1 - Count total packets

**Command:**
tcpdump -n -r dns.pcap | wc -l

**Purpose:**
Count total number of packets in the capture file

**Result:**
7

**Finding:**
The pcap contains 7 packets total - a small focused DNS capture

---

## Command 2 - Display all packets with line numbers

**Command:**
tcpdump -n -r dns.pcap -c 20 -#

**Purpose:**
Get an overview of all packets with timestamps and line numbers

**Result:**
1  15:59:15.346584 IP 4.2.2.1.53 > 172.16.16.154.57434: 45323 2/0/0 CNAME redir.espn.gns.go.com., A 68.71.212.158 (78)
2  15:59:15.558135 IP 4.2.2.1.53 > 172.16.16.154.22689: 62548 2/0/0 CNAME espn.gns.go.com., A 199.181.133.61 (68)
3  15:59:15.846451 IP 4.2.2.1.53 > 172.16.16.154.52723: 23958 3/0/0 CNAME wac.946A.edgecastcdn.net., CNAME gp1.wac.v2cdn.net., A 72.21.91.8 (118)
4  15:59:15.847604 IP 4.2.2.1.53 > 172.16.16.154.23201: 51903 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)
5  15:59:15.898379 IP 4.2.2.1.53 > 172.16.16.154.57920: 59318 4/0/0 CNAME assets.espn.go.com.edgesuite.net., CNAME a1589.g.akamai.net., A 69.31.75.194, A 69.31.75.203 (143)
6  15:59:15.979495 IP 4.2.2.1.53 > 172.16.16.154.49283: 49060 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.83, A 72.246.56.35 (135)
7  15:59:15.983886 IP 4.2.2.1.53 > 172.16.16.154.21916: 9952 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)

**Finding:**
All 7 packets are DNS responses from server 4.2.2.1 to client 172.16.16.154.
All occurred within 1 second - consistent with a single web page load
triggering multiple DNS lookups simultaneously.

---

## Command 3 - Identify all unique IPs and frequency

**Command:**
tcpdump -n -r dns.pcap | awk '{print $3}' | cut -d'.' -f1-4 | sort | uniq -c | sort -rn

**Purpose:**
Extract all source IPs, count how many times each appeared,
sort by highest frequency first

**Result:**
7 4.2.2.1

**Finding:**
Only one source IP in the entire capture.
4.2.2.1 is a Level 3 Communications public DNS server.
All 7 packets originated from this server responding to client 172.16.16.154.

---

## Command 4 - Filter DNS traffic using BPF

**Command:**
tcpdump -n -r dns.pcap 'udp port 53' -#

**Purpose:**
Use Berkeley Packet Filter BPF to isolate only UDP port 53 DNS traffic

**Result:**
1  15:59:15.346584 IP 4.2.2.1.53 > 172.16.16.154.57434: 45323 2/0/0 CNAME redir.espn.gns.go.com., A 68.71.212.158 (78)
2  15:59:15.558135 IP 4.2.2.1.53 > 172.16.16.154.22689: 62548 2/0/0 CNAME espn.gns.go.com., A 199.181.133.61 (68)
3  15:59:15.846451 IP 4.2.2.1.53 > 172.16.16.154.52723: 23958 3/0/0 CNAME wac.946A.edgecastcdn.net., CNAME gp1.wac.v2cdn.net., A 72.21.91.8 (118)
4  15:59:15.847604 IP 4.2.2.1.53 > 172.16.16.154.23201: 51903 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)
5  15:59:15.898379 IP 4.2.2.1.53 > 172.16.16.154.57920: 59318 4/0/0 CNAME assets.espn.go.com.edgesuite.net., CNAME a1589.g.akamai.net., A 69.31.75.194, A 69.31.75.203 (143)
6  15:59:15.979495 IP 4.2.2.1.53 > 172.16.16.154.49283: 49060 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.83, A 72.246.56.35 (135)
7  15:59:15.983886 IP 4.2.2.1.53 > 172.16.16.154.21916: 9952 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)

**Finding:**
BPF filter confirmed all 7 packets are DNS UDP port 53.
No other protocols are present in this capture.

---

## Command 5 - Full verbose packet details

**Command:**
tcpdump -n -r dns.pcap -v

**Purpose:**
Display full packet details including TTL, flags, protocol,
DNS query names and resolved IP addresses

**Result:**
15:59:15.346584 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 106)
4.2.2.1.53 > 172.16.16.154.57434: www.espn.com. CNAME redir.espn.gns.go.com., A 68.71.212.158

15:59:15.558135 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 96)
4.2.2.1.53 > 172.16.16.154.22689: espn.go.com. CNAME espn.gns.go.com., A 199.181.133.61

15:59:15.846451 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 146)
4.2.2.1.53 > 172.16.16.154.52723: cdn.optimizely.com. CNAME wac.946A.edgecastcdn.net., A 72.21.91.8

15:59:15.847604 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 163)
4.2.2.1.53 > 172.16.16.154.23201: a1.espncdn.com. CNAME a.espncdn.com.edgesuite.net., A 72.246.56.35, A 72.246.56.83

15:59:15.898379 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 171)
4.2.2.1.53 > 172.16.16.154.57920: assets.espn.go.com. CNAME assets.espn.go.com.edgesuite.net., A 69.31.75.194, A 69.31.75.203

15:59:15.979495 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 163)
4.2.2.1.53 > 172.16.16.154.49283: a2.espncdn.com. CNAME a.espncdn.com.edgesuite.net., A 72.246.56.83, A 72.246.56.35

15:59:15.983886 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 163)
4.2.2.1.53 > 172.16.16.154.21916: a4.espncdn.com. CNAME a.espncdn.com.edgesuite.net., A 72.246.56.35, A 72.246.56.83

**Finding:**
Verbose output revealed the actual domain names being queried.
- www.espn.com resolved to 68.71.212.158
- espn.go.com resolved to 199.181.133.61
- cdn.optimizely.com resolved via Edgecast CDN to 72.21.91.8
- a1, a2, a4.espncdn.com all resolved via Akamai CDN
- TTL of 54 on all packets - consistent responses from same DNS server
- Flags DF Don't Fragment on all packets - normal DNS behavior

---

## Summary of Findings

| Item                | Value                                              |
|---------------------|----------------------------------------------------|
| Total Packets       | 7                                                  |
| Capture Duration    | 637 milliseconds                                   |
| DNS Server          | 4.2.2.1 Level 3 Public DNS                        |
| Client IP           | 172.16.16.154                                      |
| Protocol            | UDP Port 53 DNS                                    |
| Domains Queried     | www.espn.com, espn.go.com, cdn.optimizely.com      |
| CDN Providers       | Akamai, Edgecast                                   |
| TTL                 | 54 on all packets                                  |
| Suspicious Activity | None detected                                      |
| Verdict             | Normal ESPN website browsing - legitimate DNS traffic |

EOF
