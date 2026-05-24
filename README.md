# Healing a Broken Network

## Overview
This project is an enterprise network modernization case study for Harshborough hospital, a 10-floor medical facility with 263 clients and 15 servers experiencing critical infrastructure limitations. The existing network relied on a single flat-topology router handling all internal and WAN traffic, with no segmentation, no redundancy, and inefficient use of public IP addressing. This project proposes and simulates a complete network redesign using Cisco Packet Tracer, covering hierarchical architecture, VLAN segmentation, VoIP, WLAN, routing, NAT/PAT, and access control, treated not as an academic exercise, but as a real enterprise architecture proposal.

## The Problem
Harsborough hospital's existing network suffered from several fundamental design flaws that directly threatened operational continuity in a healthcare environment where network downtime is not an option. The entire network was built around a single core router handling both internal routing and WAN connectivity, meaning all inter-floor traffic, server access, and internet requests were funneled through one device, creating a severe bottleneck that worsened as hospital digital services expanded.

Beyond the centralized bottleneck, the network had no redundancy at any layer. A single link connected each floor to the core router, meaning any link failure would completely isolate the affected floor from the rest of the network including access to electronic health records, radiology systems, and patient information. This level of single point of failure is unacceptable in a clinical environment where delayed access to patient data carries direct consequences for patient safety.

The IP addressing scheme compounded the problem further. Every floor was assigned a full public class C block of 254 addresses despite actual device counts ranging from 14 to 43 per floor, resulting in an effective utilization rate of only 10.9% accross 2,540 allocated addresses for 278 actual devices. The use of public IP addresses for internal devices also created unnecessary dependency on ISP-rented address blocks, driving up operational costs without any technical justification. Finally, that flat per-floor segmentation provided no functional isolation between servers, clinical devices, staff workstations, and wireless clients, making access control, QoS enforcement, and security policy implementation effectively impossible.

## Proposed Architecture
The proposed architecture replaces the flat single-route topology with a two-tier hierarchical design consisting of a core/distribution layer and an access layer, built around the principle that routing complexity should be handled at the center while the edges remain simple and consistent. A Cisco 2811 router sits at the top of the hierarchy as the exclusive WAN gatway, handling NAT/PAT and ISP connectivity without any involvement in internal routing decisions. Below it, a Cisco 3650-24PS Layer 3 switch servers as the core/distribution layer, taking full ownership of inter-VLAN routing via Switch Virtual Interfaces and running single-area OSPF with the router for dynamic route exchange.

Three access switches sit at the bottom of the hierarchy, each serving a dedicated segment of the network. The Servers Switch connects the 15-server data center clust along with the dedicated DHCP server and Network Controller. The Floor 1 to 5 Switch serves administrative and operational staff with wired access and VoIP. The Floor 6 to 10 Switch serves the inpatient department with clinical workstations, IP phones, and wireless access point for staff and guest WLAN coverage. All backbone connections between the core switch and access switch use redundant gigabit uplinks managed by Spanning Tree Protocol, providing automatic failover in the event of a link failure without manual intervention.

The router is deliberately kept out of internal routing decisions. All inter-VLAN traffic is handled locally by the Layer 3 switch, meaning clinical staff accessing electronic health records, radiology systems, or patient information never experience the latency of traversing a WAN-edge device for internal requests. The router only processes traffic that is genuinely destined for the internet, which is then translated via PAT to a single public IP address before leaving the network, eliminating the need for multiple rented public address blocks from the ISP.

## Key Design Decisions
## Technologies Used
## Network Topology
## VLAN Design & IP Addressing
## Configuration Highlights
## Simulation Limitations
## Future Considerations
## Lessons Learned
