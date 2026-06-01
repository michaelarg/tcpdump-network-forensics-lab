# TCPdump Commands Log
## Project: DNS Pcap Investigation
## Analyst: Michael Argaw
## Date: June 1, 2026
## Environment: Kali Linux

---

## Command 1 — Read pcap and count total packets
```bash
tcpdump -n -r dns.pcap | wc -l
```
**Purpose:** Count total number of packets in the capture file  
**Result:** reading from file dns.pcap, link-type EN10MB (Ethernet), snapshot length 262144
7

---

## Command 2 — Display first 20 packets with line numbers
```bash
tcpdump -n -r dns.pcap -c 20 -#
```
**Purpose:** Get an overview of the first 20 packets  
**Result:** reading from file dns.pcap, link-type EN10MB (Ethernet), snapshot length 262144
    1  15:59:15.346584 IP 4.2.2.1.53 > 172.16.16.154.57434: 45323 2/0/0 CNAME redir.espn.gns.go.com., A 68.71.212.158 (78)
    2  15:59:15.558135 IP 4.2.2.1.53 > 172.16.16.154.22689: 62548 2/0/0 CNAME espn.gns.go.com., A 199.181.133.61 (68)
    3  15:59:15.846451 IP 4.2.2.1.53 > 172.16.16.154.52723: 23958 3/0/0 CNAME wac.946A.edgecastcdn.net., CNAME gp1.wac.v2cdn.net., A 72.21.91.8 (118)
    4  15:59:15.847604 IP 4.2.2.1.53 > 172.16.16.154.23201: 51903 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)
    5  15:59:15.898379 IP 4.2.2.1.53 > 172.16.16.154.57920: 59318 4/0/0 CNAME assets.espn.go.com.edgesuite.net., CNAME a1589.g.akamai.net., A 69.31.75.194, A 69.31.75.203 (143)
    6  15:59:15.979495 IP 4.2.2.1.53 > 172.16.16.154.49283: 49060 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.83, A 72.246.56.35 (135)
    7  15:59:15.983886 IP 4.2.2.1.53 > 172.16.16.154.21916: 9952 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)


---

## Command 3 — Identify all unique IPs and frequency
```bash
tcpdump -n -r dns.pcap | awk '{print $3}' | cut -d'.' -f1-4 | sort | uniq -c | sort -rn
```
**Purpose:** Find all unique source IPs and how many times each appeared
**Result:** reading from file dns.pcap, link-type EN10MB (Ethernet), snapshot length 262144
      7 4.2.2.1  
**Finding:** Only one IP found — 4.2.2.1 (Level 3 public DNS server) appeared 7 times

---

## Command 4 — Show all DNS traffic
```bash
tcpdump -n -r dns.pcap 'udp port 53' -#
```
**Purpose:** Filter and display only DNS packets  
**Result:** reading from file dns.pcap, link-type EN10MB (Ethernet), snapshot length 262144
    1  15:59:15.346584 IP 4.2.2.1.53 > 172.16.16.154.57434: 45323 2/0/0 CNAME redir.espn.gns.go.com., A 68.71.212.158 (78)
    2  15:59:15.558135 IP 4.2.2.1.53 > 172.16.16.154.22689: 62548 2/0/0 CNAME espn.gns.go.com., A 199.181.133.61 (68)
    3  15:59:15.846451 IP 4.2.2.1.53 > 172.16.16.154.52723: 23958 3/0/0 CNAME wac.946A.edgecastcdn.net., CNAME gp1.wac.v2cdn.net., A 72.21.91.8 (118)
    4  15:59:15.847604 IP 4.2.2.1.53 > 172.16.16.154.23201: 51903 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)
    5  15:59:15.898379 IP 4.2.2.1.53 > 172.16.16.154.57920: 59318 4/0/0 CNAME assets.espn.go.com.edgesuite.net., CNAME a1589.g.akamai.net., A 69.31.75.194, A 69.31.75.203 (143)
    6  15:59:15.979495 IP 4.2.2.1.53 > 172.16.16.154.49283: 49060 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.83, A 72.246.56.35 (135)
    7  15:59:15.983886 IP 4.2.2.1.53 > 172.16.16.154.21916: 9952 4/0/0 CNAME a.espncdn.com.edgesuite.net., CNAME a1589.g1.akamai.net., A 72.246.56.35, A 72.246.56.83 (135)


---

## Command 5 — Verbose output to see full packet details
```bash
tcpdump -n -r dns.pcap -v
```
**Purpose:** See full details including TTL, DNS query names, and response data  
**Result:** reading from file dns.pcap, link-type EN10MB (Ethernet), snapshot length 262144
15:59:15.346584 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 106)
    4.2.2.1.53 > 172.16.16.154.57434: 45323 2/0/0 www.espn.com. CNAME redir.espn.gns.go.com., redir.espn.gns.go.com. A 68.71.212.158 (78)
15:59:15.558135 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 96)
    4.2.2.1.53 > 172.16.16.154.22689: 62548 2/0/0 espn.go.com. CNAME espn.gns.go.com., espn.gns.go.com. A 199.181.133.61 (68)
15:59:15.846451 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 146)
    4.2.2.1.53 > 172.16.16.154.52723: 23958 3/0/0 cdn.optimizely.com. CNAME wac.946A.edgecastcdn.net., wac.946A.edgecastcdn.net. CNAME gp1.wac.v2cdn.net., gp1.wac.v2cdn.net. A 72.21.91.8 (118)
15:59:15.847604 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 163)
    4.2.2.1.53 > 172.16.16.154.23201: 51903 4/0/0 a1.espncdn.com. CNAME a.espncdn.com.edgesuite.net., a.espncdn.com.edgesuite.net. CNAME a1589.g1.akamai.net., a1589.g1.akamai.net. A 72.246.56.35, a1589.g1.akamai.net. A 72.246.56.83 (135)
15:59:15.898379 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 171)
    4.2.2.1.53 > 172.16.16.154.57920: 59318 4/0/0 assets.espn.go.com. CNAME assets.espn.go.com.edgesuite.net., assets.espn.go.com.edgesuite.net. CNAME a1589.g.akamai.net., a1589.g.akamai.net. A 69.31.75.194, a1589.g.akamai.net. A 69.31.75.203 (143)
15:59:15.979495 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 163)
    4.2.2.1.53 > 172.16.16.154.49283: 49060 4/0/0 a2.espncdn.com. CNAME a.espncdn.com.edgesuite.net., a.espncdn.com.edgesuite.net. CNAME a1589.g1.akamai.net., a1589.g1.akamai.net. A 72.246.56.83, a1589.g1.akamai.net. A 72.246.56.35 (135)
15:59:15.983886 IP (tos 0x0, ttl 54, id 0, offset 0, flags [DF], proto UDP (17), length 163)
    4.2.2.1.53 > 172.16.16.154.21916: 9952 4/0/0 a4.espncdn.com. CNAME a.espncdn.com.edgesuite.net., a.espncdn.com.edgesuite.net. CNAME a1589.g1.akamai.net., a1589.g1.akamai.net. A 72.246.56.35, a1589.g1.akamai.net. A 72.246.56.83 (135)

---

