# Subnet & Inter-VLAN Routing Practice Lab

A Cisco Packet Tracer practical lab demonstrating subnetting design, IP addressing schemes, and inter-subnet / inter-VLAN routing across network segments.

---

## 🎯 Overview

In modern enterprise networks, dividing a network into smaller subnets improves performance, enhances security segmentation, and optimizes address space allocation. This lab focuses on practicing IP subnetting and configuring routing mechanisms to enable seamless communication between different subnets.

### Key Objectives
- **Subnet Planning & IP Allocation:** Design and assign subnet masks, network addresses, usable host ranges, and default gateways.
- **Inter-Subnet Communication:** Configure routing (Router-on-a-Stick / Layer 3 routing / static routing) to allow hosts in different subnets/VLANs to communicate.
- **End-to-End Connectivity:** Verify end-to-end traffic flow between segmented hosts using ICMP (`ping`) and route tracing (`traceroute`).
- **Gateway & Interface Configuration:** Configure appropriate IP addresses on router/switch interfaces and default gateways on end-user hosts.

---

## 📂 Lab Files

- [`subnet-inter-routing.pkt`](file:///Users/konquest/code_repo/network-portfolio/subnet-routing/subnet-inter-routing.pkt): Cisco Packet Tracer topology and configuration file.

---

## 🛠️ Prerequisites & Tools

- **Simulator:** [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (v8.0+ recommended)
- **Concepts Covered:**
  - IPv4 Addressing & Subnetting (CIDR / VLSM)
  - Default Gateways & Host IP settings
  - Inter-VLAN / Inter-Subnet Routing
  - Routing tables & interface status verification

---

## 🚀 How to Run the Lab

1. Launch **Cisco Packet Tracer**.
2. Open [`subnet-inter-routing.pkt`](file:///Users/konquest/code_repo/network-portfolio/subnet-routing/subnet-inter-routing.pkt) via `File > Open`.
3. Inspect device addressing and routing table entries:
   ```cisco
   enable
   show ip interface brief
   show ip route
   ```
4. Test connectivity from end devices across subnets:
   ```bash
   ping <destination-ip>
   traceroute <destination-ip>
   ```
