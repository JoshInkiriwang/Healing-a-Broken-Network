# Healing a Broken Network

## Overview
This project is an enterprise network modernization case study for Harshborough hospital, a 10-floor medical facility with 263 clients and 15 servers experiencing critical infrastructure limitations. The existing network relied on a single flat-topology router handling all internal and WAN traffic, with no segmentation, no redundancy, and inefficient use of public IP addressing. This project proposes and simulates a complete network redesign using Cisco Packet Tracer, covering hierarchical architecture, VLAN segmentation, VoIP, WLAN, routing, NAT/PAT, and access control, treated not as an academic exercise, but as a real enterprise architecture proposal.

## The Problem
Harsborough hospital's existing network suffered from several fundamental design flaws that directly threatened operational continuity in a healthcare environment where network downtime is not an option. The entire network was built around a single core router handling both internal routing and WAN connectivity, meaning all inter-floor traffic, server access, and internet requests were funneled through one device, creating a severe bottleneck that worsened as hospital digital services expanded.

Beyond the centralized bottleneck, the network had no redundancy at any layer. A single link connected each floor to the core router, meaning any link failure would completely isolate the affected floor from the rest of the network including access to electronic health records, radiology systems, and patient information. This level of single point of failure is unacceptable in a clinical environment where delayed access to patient data carries direct consequences for patient safety.

The IP addressing scheme compounded the problem further. Every floor was assigned a full public class C block of 254 addresses despite actual device counts ranging from 14 to 43 per floor, resulting in an effective utilization rate of only 10.9% accross 2,540 allocated addresses for 278 actual devices. The use of public IP addresses for internal devices also created unnecessary dependency on ISP-rented address blocks, driving up operational costs without any technical justification. Finally, that flat per-floor segmentation provided no functional isolation between servers, clinical devices, staff workstations, and wireless clients, making access control, QoS enforcement, and security policy implementation effectively impossible.

## Proposed Architecture
## Key Design Decisions
## Technologies Used
## Network Topology
## VLAN Design & IP Addressing
## Configuration Highlights
## Simulation Limitations
## Future Considerations
## Lessons Learned
