# pfSense Network Firewall & DoS Mitigation Home Lab

## Overview
I built this virtual lab in VMware Workstation to test how a stateful perimeter firewall (pfSense CE 2.7.2) handles Denial-of-Service (DoS) floods in real time. The setup places a Kali Linux machine on an external WAN segment (`192.168.71.0/24`) and an Ubuntu Desktop machine on an isolated internal LAN (`192.168.1.0/24`) behind pfSense.

During the build, I also had to troubleshoot a routing conflict caused by my local network already using `192.168.1.0/24`, configure persistent static routing in Kali, analyze ICMP and TCP SYN floods (`hping3`) in Wireshark, and configure WAN firewall rules to drop the attack at the perimeter.

## Lab Network Setup

| Virtual Machine | Role | Interface | IP Address | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Kali Linux** | External Attacker | `eth0` | `192.168.71.160/24` | VMware NAT. Static route for `192.168.1.0/24` via `192.168.71.159` |
| **pfSense 2.7.2** | Firewall / Router | `em0` (WAN) / `em1` (LAN) | `192.168.71.159/24` / `192.168.1.1/24` | Outbound NAT + LAN DHCP server enabled |
| **Ubuntu Desktop** | Internal Target | `ens33` | `192.168.1.100/24` | Isolated internal LAN segment (Gateway: `192.168.1.1`) |

## Key Steps & Commands Used

### 1. Fixing the Routing Path on Kali Linux
Initially, traffic from Kali to `192.168.1.100` went to the default VMware NAT gateway (`192.168.71.2`) instead of pfSense. I added a static route pointing `192.168.1.0/24` to the pfSense WAN IP (`192.168.71.159`) and saved it in NetworkManager so it persists across reboots:
```bash
sudo ip route replace 192.168.1.0/24 via 192.168.71.159 dev eth0
traceroute -n -m 3 192.168.1.100
