# Mini WAN Emulator Lab

A Cisco Packet Tracer implementation simulating an enterprise/service-provider Wide Area Network (WAN) core. This lab models multi-site branch/HQ interconnectivity through an ISP core mesh using **OSPF dynamic routing**, static routing at edge customer premises equipment (CPE), point-to-point `/30` links, and hosted infrastructure services (DNS/Server).

---

## Topology Overview

![WAN Emulator Topology](image.png)

```
                            [ Server-PT ]
                             (ISP Server / DNS)
                                  | 40.0.0.2/30
                                  |
                                  | 40.0.0.1/30
                            [ ISP core ]
                           /            \
             20.0.0.1/30  /              \  10.0.0.1/30
                         /   OSPF Core    \
             20.0.0.2/30/   Area 0 Mesh   \ 10.0.0.2/30
              [ Edge router 1 ] ------- [ Edge router 2 ]
                     |      30.0.0.1/30   30.0.0.2/30  |
         50.0.0.1/30 |                                 | 60.0.0.1/30
                     |                                 |
         50.0.0.2/30 |                                 | 60.0.0.2/30
             [ Central router ]                  [ Home router ]
             (1941 - HQ / Site A)                (1941 - Branch / Site B)
                     | .254                            | .254
                     | 192.168.1.0/24                  | 192.168.2.0/24
                     |                                 |
              [ Switch2 (2960) ]                [ Switch3 (2960) ]
                     |                                 |
                 LAN Site A                        LAN Site B
```

---

## Project Overview & Objectives

In real-world networks, local area networks (LANs) across geographically separated offices do not connect with direct physical cables; instead, they traverse intermediate Service Provider (ISP) WAN backbones. 

This project simulates a **Mini WAN Core** connecting two customer sites:
- **Site A (Central / HQ):** Managed by `Central router` serving local subnet `192.168.1.0/24`.
- **Site B (Home / Branch Office):** Managed by `Home router` serving local subnet `192.168.2.0/24`.
- **WAN Emulation Backbone:** A 3-router triangular mesh (`ISP core`, `Edge router 1`, `Edge router 2`) running **OSPF (Open Shortest Path First)** to emulate dynamic provider routing, path redundancy, and fault tolerance.
- **Hosted ISP Service:** An ISP-hosted DNS/application server (`ISP server` at `40.0.0.2`) accessible from all client LANs.

### Key Implementation Highlights
- **Hierarchical Network Architecture:** Clean separation of Customer Premises Equipment (CPE / Edge LANs) and Provider (P/PE) core infrastructure.
- **Dynamic Interior Gateway Protocol (OSPF Area 0):** Deployed across the ISP triangle (`ISP core`, `Edge router 1`, and `Edge router 2`) for fast convergence, optimal path selection, and automatic failover.
- **Static & Default Routing at Customer Edges:** Customer edge routers (`Central router` and `Home router`) maintain lightweight routing tables using default static routes (`0.0.0.0/0`) directed upstream toward the provider edge, reflecting standard real-world ISP handoff architecture.
- **Point-to-Point WAN Subnets (`/30` Mask):** Utilizes `255.255.255.252` masks for all inter-router serial/routed links, conserving IPv4 space.
- **Centralized Service Access:** Central DNS/Server infrastructure located off the `ISP core` router reachable across the WAN mesh from both remote branches.

---

## Network Addressing Scheme

| Network / Link Segment | Subnet | Subnet Mask | Device / Interface | Assigned IP | Description |
|:---|:---:|:---:|:---|:---:|:---|
| **Core Link: ISP Core <-> Edge Router 1** | `20.0.0.0/30` | `255.255.255.252` | `ISP core`<br>`Edge router 1` | `20.0.0.1`<br>`20.0.0.2` | ISP internal backbone link |
| **Core Link: ISP Core <-> Edge Router 2** | `10.0.0.0/30` | `255.255.255.252` | `ISP core`<br>`Edge router 2` | `10.0.0.1`<br>`10.0.0.2` | ISP internal backbone link |
| **Core Link: Edge Router 1 <-> Edge Router 2** | `30.0.0.0/30` | `255.255.255.252` | `Edge router 1`<br>`Edge router 2` | `30.0.0.1`<br>`30.0.0.2` | ISP peer transit link (redundant path) |
| **ISP Server Link** | `40.0.0.0/30` | `255.255.255.252` | `ISP core`<br>`ISP server` | `40.0.0.1`<br>`40.0.0.2` | Hosted Server / DNS uplink |
| **Customer Handoff: Site A** | `50.0.0.0/30` | `255.255.255.252` | `Edge router 1`<br>`Central router` | `50.0.0.1`<br>`50.0.0.2` | ISP PE-CE handoff to Central Site |
| **Customer Handoff: Site B** | `60.0.0.0/30` | `255.255.255.252` | `Edge router 2`<br>`Home router` | `60.0.0.1`<br>`60.0.0.2` | ISP PE-CE handoff to Home/Branch Site |
| **Central Site LAN (Switch2)** | `192.168.1.0/24` | `255.255.255.0` | `Central router`<br>Host range | `192.168.1.254`<br>`192.168.1.1 - .253` | HQ / Central office LAN gateway |
| **Home Site LAN (Switch3)** | `192.168.2.0/24` | `255.255.255.0` | `Home router`<br>Host range | `192.168.2.254`<br>`192.168.2.1 - .253` | Branch / Home office LAN gateway |

---

## Routing Architecture & Design Decisions

### 1. Dynamic Routing in the Provider Core (OSPF Area 0)
The three WAN emulation routers form a fully redundant triangular core. **OSPF** is selected for the core because:
- **Link-State Awareness & Metric Calculation:** Uses cost based on bandwidth rather than raw hop count (like RIP), ensuring traffic picks the highest-speed path.
- **Fast Convergence & Path Redundancy:** If the direct link between `Edge router 1` and `Edge router 2` (`30.0.0.0/30`) fails, OSPF automatically converges and routes traffic through `ISP core` (`20.0.0.0/30` and `10.0.0.0/30`) without administrative intervention.
- **Loop-Free Topology:** The Shortest Path First (Dijkstra SPF) algorithm prevents routing loops within the multi-path meshed core.

### 2. Static / Default Routing at Customer Edges (CPE)
Rather than running heavy dynamic routing protocols to customer edge routers:
- `Central router` and `Home router` utilize a **default static route (`ip route 0.0.0.0 0.0.0.0 <next-hop>`)** pointed to their respective ISP edge interfaces (`50.0.0.1` and `60.0.0.1`).
- This minimizes CPU and memory overhead on branch/edge routers and prevents customer LAN topologies from injecting internal state into the ISP backbone.
- ISP edge routers redistribute or statically route customer LAN prefixes (`192.168.1.0/24` and `192.168.2.0/24`) back into the core.

---

## Configuration Reference

### 1. ISP Core Router Configuration (OSPF & Server Link)
```cisco
ISP-core(config)# interface gig0/0
ISP-core(config-if)# ip address 20.0.0.1 255.255.255.252
ISP-core(config-if)# no shutdown

ISP-core(config)# interface gig0/1
ISP-core(config-if)# ip address 10.0.0.1 255.255.255.252
ISP-core(config-if)# no shutdown

ISP-core(config)# interface gig0/2
ISP-core(config-if)# ip address 40.0.0.1 255.255.255.252
ISP-core(config-if)# no shutdown

ISP-core(config)# router ospf 1
ISP-core(config-router)# router-id 1.1.1.1
ISP-core(config-router)# network 20.0.0.0 0.0.0.3 area 0
ISP-core(config-router)# network 10.0.0.0 0.0.0.3 area 0
ISP-core(config-router)# network 40.0.0.0 0.0.0.3 area 0
```

### 2. Edge Router 1 Configuration (Core OSPF + Customer Interconnect)
```cisco
Edge-router-1(config)# interface gig0/0
Edge-router-1(config-if)# ip address 20.0.0.2 255.255.255.252
Edge-router-1(config-if)# no shutdown

Edge-router-1(config)# interface gig0/1
Edge-router-1(config-if)# ip address 30.0.0.1 255.255.255.252
Edge-router-1(config-if)# no shutdown

Edge-router-1(config)# interface gig0/2
Edge-router-1(config-if)# ip address 50.0.0.1 255.255.255.252
Edge-router-1(config-if)# no shutdown

! OSPF on ISP links
Edge-router-1(config)# router ospf 1
Edge-router-1(config-router)# router-id 2.2.2.2
Edge-router-1(config-router)# network 20.0.0.0 0.0.0.3 area 0
Edge-router-1(config-router)# network 30.0.0.0 0.0.0.3 area 0
Edge-router-1(config-router)# network 50.0.0.0 0.0.0.3 area 0

! Route to Site A LAN
Edge-router-1(config)# ip route 192.168.1.0 255.255.255.0 50.0.0.2
Edge-router-1(config)# router ospf 1
Edge-router-1(config-router)# redistribute static subnets
```

### 3. Edge Router 2 Configuration (Core OSPF + Customer Interconnect)
```cisco
Edge-router-2(config)# interface gig0/0
Edge-router-2(config-if)# ip address 10.0.0.2 255.255.255.252
Edge-router-2(config-if)# no shutdown

Edge-router-2(config)# interface gig0/1
Edge-router-2(config-if)# ip address 30.0.0.2 255.255.255.252
Edge-router-2(config-if)# no shutdown

Edge-router-2(config)# interface gig0/2
Edge-router-2(config-if)# ip address 60.0.0.1 255.255.255.252
Edge-router-2(config-if)# no shutdown

! OSPF on ISP links
Edge-router-2(config)# router ospf 1
Edge-router-2(config-router)# router-id 3.3.3.3
Edge-router-2(config-router)# network 10.0.0.0 0.0.0.3 area 0
Edge-router-2(config-router)# network 30.0.0.0 0.0.0.3 area 0
Edge-router-2(config-router)# network 60.0.0.0 0.0.0.3 area 0

! Route to Site B LAN
Edge-router-2(config)# ip route 192.168.2.0 255.255.255.0 60.0.0.2
Edge-router-2(config)# router ospf 1
Edge-router-2(config-router)# redistribute static subnets
```

### 4. Customer Edge Routers (CPE Static Default Routing)

#### Central Router (Site A / HQ)
```cisco
Central-router(config)# interface gig0/0
Central-router(config-if)# ip address 50.0.0.2 255.255.255.252
Central-router(config-if)# no shutdown

Central-router(config)# interface gig0/1
Central-router(config-if)# ip address 192.168.1.254 255.255.255.0
Central-router(config-if)# no shutdown

! Default route upstream to ISP Edge Router 1
Central-router(config)# ip route 0.0.0.0 0.0.0.0 50.0.0.1
```

#### Home Router (Site B / Branch)
```cisco
Home-router(config)# interface gig0/0
Home-router(config-if)# ip address 60.0.0.2 255.255.255.252
Home-router(config-if)# no shutdown

Home-router(config)# interface gig0/1
Home-router(config-if)# ip address 192.168.2.254 255.255.255.0
Home-router(config-if)# no shutdown

! Default route upstream to ISP Edge Router 2
Home-router(config)# ip route 0.0.0.0 0.0.0.0 60.0.0.1
```

---

## Verification & Troubleshooting Commands

### 1. OSPF Neighbor Adjacencies
Verify that OSPF neighbor relationships are established in the `FULL` state across all ISP backbone links:
```cisco
ISP-core# show ip ospf neighbor
Edge-router-1# show ip ospf neighbor
Edge-router-2# show ip ospf neighbor
```

### 2. Routing Tables Inspection
Verify OSPF routes (`O` / `O E2`) and static routes (`S` / `S*`):
```cisco
ISP-core# show ip route
Central-router# show ip route
Home-router# show ip route
```

#### OSPF Route Tables Across WAN Core Routers:

- **ISP Core (`sh ip route`):**
  ![ISP Core OSPF Route Table](isp-core-ospf.png)

- **Edge Router 1 (`sh ip route`):**
  ![Edge Router 1 OSPF Route Table](edge-router1-ospf.png)

- **Edge Router 2 (`sh ip route`):**
  ![Edge Router 2 OSPF Route Table](edge-router2-ospf.png)

### 3. Interface Status & IP Assignment
```cisco
Router# show ip interface brief
```

### 4. End-to-End Connectivity Verification
- **Host Site A -> Host Site B:**
  ```bash
  ping 192.168.2.1
  traceroute 192.168.2.1
  ```
- **Host Site A / B -> ISP Server (DNS):**
  ```bash
  ping 40.0.0.2
  ```
- **WAN Link Failover Test:**
  Shut down interface `gig0/1` on `Edge router 1` (breaking link `30.0.0.0/30`). Run a continuous ping from Site A to Site B and observe OSPF rerouting traffic through `ISP core` (`20.0.0.1` -> `10.0.0.2`) with minimal packet drop.

---

## Troubleshooting and Security Review

During lab implementation and validation, ping requests between remote branch endpoints and across the WAN core initially failed. A systematic diagnostic review revealed two primary routing configuration issues:

### 1. Missing Default Static Routes on Customer Edge Routers (Central & Home)
- **Symptom:** Hosts in Site A (`192.168.1.0/24`) were unable to reach hosts in Site B (`192.168.2.0/24`) or the external ISP DNS server (`40.0.0.2`).
- **Root Cause:** Both the `Central router` and `Home router` (acting as CPE devices) lacked a quad-zero default static route (`0.0.0.0 0.0.0.0`). When packets with destination IPs outside the local LAN arrived at the customer router, the router had no matching entry in its routing table and dropped the packets (`unreachable destination`).
- **Resolution:** Configured upstream default routes pointing to the respective provider edge router interfaces:
  ```cisco
  ! On Central Router:
  Central-router(config)# ip route 0.0.0.0 0.0.0.0 50.0.0.1

  ! On Home Router:
  Home-router(config)# ip route 0.0.0.0 0.0.0.0 60.0.0.1
  ```

### 2. Missing OSPF Routes for Public Interconnect IP Subnets on Edge Routers
- **Symptom:** Even after setting default routes on customer routers, end-to-end ping between the Central and Home routers remained unsuccessful.
- **Root Cause:** In the provider edge routers (`Edge router 1` and `Edge router 2`), the public handoff subnets allocated to the customer edge connections (`50.0.0.0/30` on Edge Router 1 and `60.0.0.0/30` on Edge Router 2) were initially not advertised into the core OSPF Area 0 process. Because the intermediate core routers (`ISP core` and peer edge routers) had no route to these handoff subnets, return traffic pathing failed and ICMP replies could not be routed back to the initiating edge.
- **Resolution:** Ensured that the edge routers had their handoff subnets included in the OSPF process (or properly redistributed/routed throughout the core), allowing all routers in the provider backbone to learn the complete path back to both customer endpoints:
  ```cisco
  ! Edge router 1 OSPF configuration:
  Edge-router-1(config-router)# network 50.0.0.0 0.0.0.3 area 0

  ! Edge router 2 OSPF configuration:
  Edge-router-2(config-router)# network 60.0.0.0 0.0.0.3 area 0
  ```

---

---

## Repository Files

- [`mini-wan-emulator.pkt`](mini-wan-emulator.pkt): Cisco Packet Tracer topology and configuration file.
- [`image.png`](image.png): Annotated topology diagram showing IP subnetting, interface designations, and routing protocols.
- [`README.md`](README.md): Lab documentation, architecture design details, and configuration walkthrough.
