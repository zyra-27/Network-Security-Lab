# 🔐 Network Security Lab

Hands-on network security lab covering reconnaissance, attack simulation, firewall configuration, NAT, and secure network design. Conducted across 6 progressive assignments on Linux-based environments.

**Tools:** `nmap` · `iptables` · `hping3` · `tcpdump` · `iptraf-ng` · `nload` · `FileZilla` · `Cisco Packet Tracer`

---

## Lab Overview

| # | Topic | Key Skills |
|---|-------|------------|
| Lab 1 | Network Reconnaissance | Host discovery, port scanning, service detection |
| Lab 2 | Attack Simulation | ICMP flood, Smurf attack, traffic analysis |
| Lab 3 | Firewall Configuration | iptables rules, access control by IP/MAC/protocol |
| Lab 4 | Network Address Translation | DNAT port forwarding, SNAT/IP masquerading |
| Lab 5 | Network Design & Asset Classification | Subnetting, VLAN segmentation, asset prioritization |
| Lab 6 | Secure Network Implementation | Router/firewall config, ACL, hospital network simulation |

---

## Lab 1 – Network Reconnaissance

**Objective:** Discover live hosts and enumerate open services on a local network without prior knowledge of target IPs.

**Tools:** `nmap`

**What was done:**
- Performed ARP ping scan (`-PR` flag) to identify live hosts on `192.168.43.0/24`
- Used multiple scan options (`-sV`, `-sS`, `-A`, `-Pn`) to detect open ports and running services
- Identified open ports: `21/tcp (FTP)`, `22/tcp (SSH)`, `80/tcp (HTTP)`
- Installed and verified HTTP (Apache2) and FTP (vsftpd) services via `nmap` and `netstat -tulpn`
- Observed how RST-flagged connections in `iptraf` indicate active port scanning

**Key insight:** `nmap -Pn` is critical when ICMP is blocked — many scans fail silently without it.

---

## Lab 2 – Attack Simulation

**Objective:** Understand attack traffic patterns by simulating real network attacks and observing them live.

**Setup:** Attacker `192.168.1.117` → Victim `192.168.1.116`

**Tools:** `hping3`, `tcpdump`, `iptraf-ng`, `nload`

**What was done:**
- Simulated ICMP flood attack using `hping3 -1 -d 1000 --flood` with 1000-byte packets
- Observed traffic spike from ~0 to **1.49 MBit/s** on victim interface via `nload`
- Simulated Smurf attack targeting broadcast address (`192.168.1.255`)
- Captured baseline and attack traffic to `.pcap` files using `tcpdump`
- Confirmed ICMP flood mitigation by dropping attacker IP via `iptables`

**Key insight:** Comparing nload before vs. during attack provides clear visual evidence of DoS impact. iptables rule order is decisive — DROP must come before ACCEPT.

---

## Lab 3 – Firewall Configuration

**Objective:** Implement layered access control using iptables to restrict traffic by IP, MAC, protocol, and connection count.

**Tools:** `iptables`, `nmap`, `FileZilla`

**What was done:**
- Restricted friend's Windows host to HTTP only (FTP blocked), own Windows granted both HTTP and FTP
- Configured ICMP reply policy: selectively allowed/blocked ping replies per host
- Limited SSH concurrent connections from specific host to max 3 (`--connlimit-above 3`)
- Whitelisted SSH access by MAC address (`14:AC:60:C9:D9:75`) — all other sources timed out
- Verified every rule with `nmap` scan after each change

**Key insight:** Defense in depth — firewall rules, MAC filtering, and connection limits work best in combination.

---

## Lab 4 – Network Address Translation (NAT)

**Objective:** Configure DNAT and SNAT to enable port forwarding and hide internal topology from external clients.

**Tools:** `iptables` (NAT table), `tcpdump`

**What was done:**
- Configured DNAT to forward external traffic on port `80` and port `10000` to internal Windows web server
- Configured SNAT/MASQUERADE so friend's browser saw firewall IP (`10.188.199.185`) instead of real client IP
- Enabled IP forwarding via `/proc/sys/net/ipv4/ip_forward`
- Verified port forwarding by accessing web server through firewall IP from external host

**Key insight:** DNAT and SNAT work together to completely hide internal topology — external clients never see real internal IPs.

---

## Lab 5 – Network Design & Asset Classification

**Objective:** Design a segmented hospital network with proper asset prioritization and subnet allocation.

**Tools:** Cisco Packet Tracer

**Asset Priority Classification:**

| Priority | Systems |
|----------|---------|
| 🔴 High | Finance App & DB, Medical Records App & DB |
| 🟡 Medium | Laboratory, Paramedical, Nursing Systems |
| 🟢 Low | Public Website, Email Server, Guest Portal |

**Subnet Design:**

| Segment | Subnet | Users |
|---------|--------|-------|
| Nursing / Lab / Paramedical | `10.2.12.0/22` | 700 |
| Guest Network | `192.168.12.0/21` | 1200 |
| Finance + Medical | `10.2.17.0/25` | 100 |
| Finance Servers | `10.5.0.0/28` | — |
| Medical Servers | `10.5.0.16/28` | — |
| DMZ (Web/Email) | `172.16.5.0/28` | — |

**Key insight:** Principle of Least Privilege — every host gets only the minimum access needed. High-value assets must have dedicated servers and dedicated firewalls.

---

## Lab 6 – Secure Network Implementation

**Objective:** Build and configure a fully segmented hospital network in Cisco Packet Tracer with VLANs, inter-VLAN routing, ACLs, and NAT.

**Tools:** Cisco Packet Tracer

**What was done:**
- Configured VLANs on core switch (FINANCE, MEDICAL, LAB) with trunk port to router
- Set up router sub-interfaces for inter-VLAN routing with 802.1Q encapsulation
- Implemented ACLs to enforce: Finance ↔ Medical = BLOCK, Guest ↔ Any Internal = BLOCK
- Configured NAT overload (PAT) for internal hosts to access internet through single public IP
- Applied firewall zone structure: WAN → DMZ → Internal Segmented Network

**Network Topology:**
```
Internet
    │
[Router - Internet]
    │
[Main Router / Firewall Router]
    ├── [DMZ Switch] ── WEB01 (172.16.5.2) · MAIL01 (172.16.5.3)
    └── [Server Switch]
            ├── FIN-APP01 (10.5.0.2) · FIN-DB01 (10.5.0.3)
            ├── MED-APP01 (10.5.0.18) · MED-DB01 (10.5.0.19)
            └── LAB-APP01 (10.5.0.34)
```

---

## Key Takeaways

Network security is not a single tool or rule — it is a **layered, cumulative discipline**.

- Every attack scenario (reconnaissance → exploitation) has a corresponding defense at each layer
- Rule order in iptables is decisive: always verify with nmap after every change
- Network segmentation with VLANs and ACLs is the foundation of enterprise security design
- DNAT + SNAT together can completely abstract internal topology from external parties

---

## Documents

| File | Description |
|------|-------------|
| [`docs/summary-report.docx`](docs/summary-report.docx) | Lab Project Summary Report (all outcomes) |
| [`docs/group-assignment.pdf`](docs/group-assignment.pdf) | Full assignment with screenshots and configurations |

---

## Group Members

- Almira Shinta Aulia
- Melinia Fransiska Telaumbanua
- Naura Rizky
- Shofiyatur Rahmatin Nazilah

*Network Security Course — 2025*
