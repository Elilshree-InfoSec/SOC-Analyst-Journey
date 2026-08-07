# Networking Fundamentals for SOC Analyst (Tier 1)

These notes cover the core networking knowledge a Tier 1 SOC analyst needs — not just definitions, but *why it matters* when you're triaging alerts, reading logs, or investigating a ticket.

---

## 1. OSI Model (7 Layers)

The OSI model is a **conceptual** framework for how data moves across a network.

Mnemonic (top to bottom): **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

| Layer | Name | What Happens Here | SOC Relevance |
|---|---|---|---|
| 7 | Application | User-facing protocols: HTTP, DNS, DHCP, FTP, SMTP | Web attacks, malicious payloads, phishing emails, C2 traffic often surface here |
| 6 | Presentation | Data formatting, encryption/decryption, compression (SSL/TLS sits around here) | TLS inspection, certificate issues, encoded payloads |
| 5 | Session | Establishes/manages/terminates sessions between hosts | Session hijacking, session fixation |
| 4 | Transport | End-to-end delivery — TCP (reliable) / UDP (fast, no guarantee) | Port scanning, SYN floods, unusual port usage |
| 3 | Network | Logical addressing & routing — IPv4, IPv6, ICMPv4/6 | IP spoofing, routing anomalies, ICMP tunneling |
| 2 | Data Link | Physical addressing (MAC), switches, ARP, VLANs, Ethernet, WLAN | ARP spoofing, MAC flooding, VLAN hopping |
| 1 | Physical | Cables, radio signals, NICs, actual bits on the wire | Rogue devices, physical tampering, unauthorized wireless APs |

**Why it matters:** When you get an alert, thinking "which layer is this happening at?" helps you quickly figure out what tool or log source to check (firewall = L3/L4, proxy/WAF = L7, switch logs = L2).

---

## 2. TCP/IP Model (4 Layers)

The practical model the real internet actually runs on. Maps to OSI like this:

| TCP/IP Layer | Maps to OSI Layers | Protocols/Examples |
|---|---|---|
| Application | Application, Presentation, Session (5-7) | HTTP, HTTPS, DNS, DHCP, FTP, SMTP |
| Transport | Transport (4) | TCP, UDP |
| Internet | Network (3) | IPv4, IPv6, ICMP |
| Network Access | Data Link, Physical (1-2) | Ethernet, WLAN, ARP |

**SOC takeaway:** Most SIEM/EDR alerts you'll triage reference Application-layer activity (URLs, domains, file hashes) combined with Internet/Transport-layer metadata (source/destination IP, port, protocol). Knowing this mapping helps you read a packet capture or log line and immediately know what's "normal" structure vs. what's off.

---

## 3. TCP vs UDP

One of the most common interview questions for a SOC Tier 1 role — know this cold.

### TCP
- Connection-oriented
- Reliable delivery
- Uses the 3-way handshake
- Error checking and retransmission

Common protocols:
- HTTP/HTTPS
- FTP
- SSH
- SMTP

### UDP
- Connectionless
- Faster
- No guarantee of delivery
- No retransmission

Common protocols:
- DNS
- DHCP
- VoIP
- Streaming

### SOC Relevance
- TCP SYN floods
- UDP floods
- DNS traffic uses UDP (mostly)
- Log analysis often requires identifying whether traffic uses TCP or UDP

---

## 4. TCP Three-Way Handshake

1. **SYN** — client sends a synchronize request to the server
2. **SYN-ACK** — server acknowledges and sends its own synchronize request back
3. **ACK** — client acknowledges, connection established

**Purpose:** Establishes a reliable TCP connection before any actual data is sent.

**SOC Relevance:**
- **SYN flood attacks** — attacker sends a flood of SYN packets without completing the handshake, exhausting server resources (a classic DoS technique)
- **Half-open connections** — connections stuck after SYN but before ACK; a spike in these is a red flag
- **Network troubleshooting** — a handshake that never completes usually points to a firewall block, routing issue, or unreachable host

---

## 5. How Data Travels (Packet Flow)

A simplified view of how data moves down the stack, across the network, and back up at the destination:

```
Application
    ↓
Transport
    ↓
Internet
    ↓
Network Access
    ↓
Destination
```

Each layer wraps the data from the layer above it (encapsulation) as it travels down, and unwraps it (de-encapsulation) as it travels up the stack at the receiving end.

**SOC Relevance:** Understanding where packets travel helps you identify where in the chain an attack is occurring and which logs to pull — e.g., a DNS-layer issue means checking DNS server logs, while a flood/scan issue means checking firewall or NetFlow data at the Transport/Internet layers.

---

## 6. IP Addressing: IPv4 vs IPv6

### IPv4
- 32-bit address, written as 4 decimal octets: `192.168.1.10`
- Address space: ~4.3 billion addresses (exhausted long ago — why NAT and IPv6 exist)
- Address classes (mostly legacy knowledge now, but still tested):
  - Class A: 1.0.0.0 – 126.255.255.255 (large networks)
  - Class B: 128.0.0.0 – 191.255.255.255 (medium networks)
  - Class C: 192.0.0.0 – 223.255.255.255 (small networks)
  - Class D: Multicast, Class E: Experimental

### IPv6
- 128-bit address, written in 8 groups of hex, separated by colons:
  `2001:0db8:85a3:0000:0000:8a2e:0370:7334`
- Can be shortened: leading zeros dropped, one run of consecutive `0000` groups replaced with `::`
  → `2001:db8:85a3::8a2e:370:7334`
- Huge address space — no more NAT-driven address conservation needed
- No broadcast — uses multicast/anycast instead
- Built-in features: IPsec support, simplified header, auto-configuration (SLAAC)

**Why IPv6 matters to a SOC analyst:**
- Many networks run **dual-stack** (IPv4 + IPv6) — attackers sometimes use IPv6 to evade detection because monitoring tools/rules are often IPv4-only. Always check if IPv6 logging is enabled.
- **6to4/Teredo tunneling** can be abused to smuggle traffic past IPv4-only firewalls — a classic evasion technique worth flagging.
- Don't assume "no IPv4 traffic" means "no traffic" — check both stacks.

---

## 7. Public vs Private IP

Private (RFC 1918) ranges are **not routable on the internet** — used inside internal networks:

| Range | CIDR | Common Use |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | Large enterprise networks |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Home/small office networks |

Public IPs are globally routable and unique on the internet.

**SOC relevance:** If you see a "private" internal IP directly reaching out to the internet without going through NAT/proxy, that's suspicious. Also — seeing internal-to-internal lateral movement between private IPs is a big deal in an incident (potential pivoting).

---

## 8. Subnetting (Basics)

Subnetting splits a larger network into smaller logical segments using a **subnet mask** or **CIDR notation** (`/24`, `/16`, etc.)

- `/24` = 255.255.255.0 → 256 addresses (254 usable) — common for a small office LAN
- `/16` = 255.255.0.0 → 65,536 addresses
- `/32` = a single host

**Quick example:**
`192.168.1.0/24` covers `192.168.1.0` – `192.168.1.255`
- Network address: `192.168.1.0`
- Broadcast address: `192.168.1.255`
- Usable hosts: `192.168.1.1` – `192.168.1.254`

**Why it matters:** You'll constantly need to answer "is this IP inside our network?" or "which subnet/segment does this alert belong to?" — that's basic CIDR math. It also helps you scope an incident (e.g., "the infected host is on the 10.10.5.0/24 finance subnet").

---

## 9. MAC Address & ARP

- **MAC Address**: A 48-bit physical hardware address burned into a NIC, written as `00:1A:2B:3C:4D:5E`. Operates at Layer 2.
- **ARP (Address Resolution Protocol)**: Maps an IP address (Layer 3) to a MAC address (Layer 2) so devices on the same local network can actually deliver frames to each other.

**Security relevance:**
- **ARP Spoofing/Poisoning**: An attacker sends fake ARP replies to associate their MAC with another device's IP (often the gateway), enabling man-in-the-middle attacks or traffic interception. Look for duplicate IP-to-MAC mappings or unexpected ARP traffic spikes.
- **MAC flooding**: Overwhelming a switch's MAC address table to force it into "hub mode," broadcasting traffic to all ports (easy sniffing).

---

## 10. DNS (Domain Name System)

Translates human-readable domain names (`example.com`) into IP addresses. Runs primarily on **UDP/TCP port 53**.

**Common attack patterns SOC analysts should recognize:**
- **DNS Tunneling**: Attackers encode data (C2 commands, exfiltrated data) inside DNS queries/responses to bypass firewalls — look for abnormally long/frequent DNS queries to unusual domains.
- **DNS Spoofing/Cache Poisoning**: Injecting false DNS records to redirect users to malicious sites.
- **Fast Flux**: Rapidly changing IPs behind a domain to evade blocklisting — common with botnets.
- **DGA (Domain Generation Algorithms)**: Malware generating pseudo-random domains for C2 — random-looking domain names are a red flag.

---

## 11. DHCP (Dynamic Host Configuration Protocol)

Automatically assigns IP addresses, subnet masks, gateways, and DNS servers to devices on a network.

**The DORA process:**
1. **D**iscover — client broadcasts for a DHCP server
2. **O**ffer — server offers an IP
3. **R**equest — client requests that IP
4. **A**cknowledge — server confirms the lease

**Security relevance:**
- **Rogue DHCP server**: An attacker sets up an unauthorized DHCP server to hand out malicious gateway/DNS settings, redirecting victim traffic through them (MITM setup).
- **DHCP starvation**: Attacker floods the DHCP server with fake requests to exhaust the IP pool, causing denial of service.

---

## 12. NAT (Network Address Translation)

Translates private internal IPs to a public IP (and back) so internal devices can communicate with the internet while staying hidden/unroutable directly.

- **Static NAT**: One private IP ↔ one public IP (fixed mapping)
- **Dynamic NAT**: Private IPs mapped to a pool of public IPs
- **PAT (Port Address Translation / "NAT overload")**: Many private IPs share one public IP, differentiated by port numbers — the most common type in home/office routers

**SOC relevance:** When investigating an alert with a public source IP, remember that IP might represent *many* internal users/devices behind NAT/PAT. You often need NAT/firewall logs to map the public IP + port back to the actual internal host and timestamp.

---

## 13. Common Protocols

| Protocol | Purpose | Port(s) | SOC Notes |
|---|---|---|---|
| HTTP | Unencrypted web traffic | 80 | Cleartext — credentials/data visible in captures; often abused for C2 |
| HTTPS | Encrypted web traffic (TLS/SSL) | 443 | Can't inspect content without TLS interception; check cert validity, SNI, JA3 fingerprints |
| FTP | File transfer (control + data channels) | 20 (data), 21 (control) | Sends credentials in cleartext by default — high risk if seen on the wire |
| SSH | Secure remote shell/login | 22 | Legitimate admin use, but also abused for tunneling/lateral movement; watch for brute-force attempts |
| SMTP | Sending email | 25 | Central to phishing investigations; check headers, SPF/DKIM/DMARC |
| POP3 | Retrieving email (downloads & often deletes from server) | 110 | Cleartext by default; credential exposure risk |
| IMAP | Retrieving email (syncs with server) | 143 | Same cleartext concern as POP3 unless using IMAPS |
| ICMP | Network diagnostics (ping, traceroute, error messages) | N/A (not port-based) | Used for recon (ping sweeps) and covert channels (ICMP tunneling/data exfil) |

---

## 14. Common Ports Cheat Sheet

| Port | Protocol/Service |
|---|---|
| 20, 21 | FTP (data, control) |
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |
| 3389 | RDP |

**Extra ones worth knowing at Tier 1 (commonly seen in real alerts):**
- 23 – Telnet (cleartext, legacy, big red flag if seen)
- 67/68 – DHCP
- 139/445 – SMB/NetBIOS (common lateral movement & ransomware spread vector — e.g., EternalBlue)
- 3306 – MySQL
- 8080/8443 – Alternate HTTP/HTTPS (proxies, web apps, sometimes malware C2)

---

## 15. Common Network Devices

**Router**
- Connects different networks and routes traffic between them.

**Switch**
- Connects devices within a LAN, forwarding traffic based on MAC address.

**Firewall**
- Filters traffic based on rules (IP, port, protocol, application).

**Proxy**
- Intermediary for web traffic; forwards requests on a client's or server's behalf.

**IDS (Intrusion Detection System)**
- Monitors traffic and detects suspicious activity — alerts only, doesn't block.

**IPS (Intrusion Prevention System)**
- Detects *and* blocks malicious traffic in real time — sits inline.

**Load Balancer** (basic)
- Distributes traffic across multiple servers to balance load and improve availability.

---

## 16. Network Security Basics

### Firewalls
Filter traffic based on rules (IP, port, protocol, and increasingly application-layer content for NGFWs). First line of defense — SOC analysts frequently pull firewall logs to confirm whether traffic was allowed or blocked.

### Proxies
Sit between clients and the internet, forwarding requests on their behalf.
- **Forward proxy**: Protects/controls outbound client traffic (common for web filtering, logging).
- **Reverse proxy**: Sits in front of servers, protecting them and load-balancing inbound traffic.
Useful for SOC: proxy logs often show full URLs and user-agents — great for tracing what a user actually accessed.

### VPN (Virtual Private Network)
Creates an encrypted tunnel between two points over an untrusted network (like the internet), commonly used for secure remote access.
- SOC relevance: legitimate VPN traffic is normal, but watch for **impossible travel** (same user logging in from two geographically distant VPN exit points close in time), unusual VPN login times, or connections from known malicious VPN/proxy exit nodes.

---

## 17. Basic Packet Structure (Optional, Useful for Wireshark)

When you open a packet capture, this is roughly what you're looking at for a typical TCP/IP packet:

- **Source IP** — where the packet came from
- **Destination IP** — where the packet is headed
- **Source Port** — the sending application's port
- **Destination Port** — the receiving application's/service's port
- **Protocol** — TCP, UDP, ICMP, etc.
- **Payload** — the actual data being carried

**SOC Relevance:** These fields are exactly what you'll filter on in Wireshark or a SIEM query (e.g., `ip.addr == x.x.x.x && tcp.port == 445`) when investigating an alert or hunting for anomalous traffic.

---

## Quick Study Priorities for Tier 1 Interviews/Job

If you only have limited time, prioritize being able to **explain out loud, from memory**:
1. The 7 OSI layers and one example protocol/attack per layer
2. TCP vs UDP (reliable vs connectionless) — very commonly asked
3. The TCP 3-way handshake (SYN, SYN-ACK, ACK) and what a SYN flood looks like
4. The DNS resolution process and at least 2 DNS-based attacks
5. Public vs private IP ranges from memory
6. The port table above — cold
7. How NAT/PAT affects log investigation (mapping public IP+port → internal host)
8. The difference between IDS and IPS

Good luck with the role — this is genuinely the right foundation to build on before diving deeper into SIEM tools, log analysis, and specific attack playbooks.
