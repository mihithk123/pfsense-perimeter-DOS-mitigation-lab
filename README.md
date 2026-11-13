# Enterprise Network Perimeter Security & DoS Mitigation Lab (pfSense, Kali Linux, Ubuntu, Wireshark)

![Security](https://img.shields.io/badge/Focus-Network%20Security%20%26%20SOC-blue)
![Firewall](https://img.shields.io/badge/Firewall-pfSense%20CE%202.7.2-orange)
![Attacker](https://img.shields.io/badge/Offense-Kali%20Linux%20%7C%20hping3-red)
![Analysis](https://img.shields.io/badge/Packet%20Analysis-Wireshark-brightgreen)

## 📌 Executive Summary
This project demonstrates an end-to-end **Purple Team Network Security Lab** built in VMware Workstation. It simulates an external threat actor (**Kali Linux**) launching volumetric **ICMP Echo Flood** and **TCP SYN Flood** Denial-of-Service (DoS) attacks against an internal corporate endpoint (**Ubuntu Desktop**) protected by a **pfSense stateful network firewall**.

Beyond configuring Access Control Lists (ACLs), this project documents real-world routing diagnostics—resolving a subnet collision between the physical host network and the isolated virtual LAN through persistent Linux NetworkManager static routing—and verifies **100% perimeter mitigation (155.64 MiB of flood traffic dropped at the WAN interface)** using live packet captures in **Wireshark**.

---

## 🏗️ Network Topology & IP Architecture

| System Role | Operating System | Interface | IP Address / Subnet | Network Segment | Gateway / Routing |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **External Threat Actor** | Kali Linux | `eth0` | `192.168.71.160/24` | VMware NAT (Simulated WAN) | Default: `192.168.71.2` + Static Route: `192.168.1.0/24 via 192.168.71.159` |
| **Perimeter Firewall** | pfSense CE 2.7.2 | `em0` (WAN)<br>`em1` (LAN) | `192.168.71.159/24`<br>`192.168.1.1/24` | VMware NAT (WAN)<br>Internal LAN Segment (`LabNet`) | Stateful Inspection, Outbound NAT, LAN DHCP Server |
| **Internal Protected Host** | Ubuntu Desktop | `ens33` | `192.168.1.100/24` | Internal LAN Segment (`LabNet`) | Default Gateway: `192.168.1.1` (pfSense LAN) |

---

## 🔧 Key Engineering & Troubleshooting Milestones

### 1. Resolving Split-Routing & Subnet Collision
* **Problem:** Because the physical host network operated on `192.168.1.0/24`, initial traffic from Kali (`192.168.71.160`) to the internal Ubuntu target (`192.168.1.100`) followed the default NAT gateway (`192.168.71.2`) and escaped to the physical host instead of reaching the pfSense WAN interface (`192.168.71.159`).
* **Diagnostic:** Identified the routing leak using `traceroute -n 192.168.1.100`.
* **Solution:** Configured an explicit static route directing all `192.168.1.0/24` traffic through the pfSense WAN interface (`192.168.71.159`) and persisted it inside Kali's **NetworkManager IPv4 Route configuration** so it survives system reboots:
  ```bash
  sudo ip route replace 192.168.1.0/24 via 192.168.71.159 dev eth0
  sudo ip route flush cache
