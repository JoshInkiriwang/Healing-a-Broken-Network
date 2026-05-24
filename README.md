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
__Keeping OSPF Instead of Replacing It__ <br>
The original case brief expressed a desire to move away from OSPF toward something easier to manage, After analysis, the conclusion was that OSPF itself was never the problem. THe real source of operational complexity was the flat topology forcing all routing through a single device without hierarchy. Rather than introducing a new protocol and its associated learning curve, the architecture was redesign around OSPF so that it runs only between the router and the core Layer 3 switch in a single-area configuration. Single-area OSPF is significantly easier to configure, troubleshoot and maintain compared to multi-area OSPF, and it is more than sufficient for a network of this scale. The result is that OSPF become simpler to manage not by replacement but by giving it a better environment to operate in

__Function-Based VLAN Segmentation instead of Floor-Based__ <br>
The original network segmented by floor location, which seems intuitive but falls in a hospital environment where clinical staff move constantly between floors and need consistent network access regardless of physical location. The proposed design segments by function instead, meaning a doctor connecting from floor 6 to floor 9 lands on the same Staff WLAN VLAN with the same access rights, the same QoS policy, and the same ACL enforcement. This approach also makes security policy significantly cleaner because each VLAN represents a homogeneous group of devices with identical access requirements, making ACL rules precise and auditable rather than broad and approximate.

__RFC1918 Private Addressing with VLSM__ <br>
Replacing 10 rented public Class C blocks with a single private 172.16.0.0/16 block eliminate ISP depedency for internal addressing entirely. VLSM allows each subnet to be sized proportionally to its actual host count while maintaining a reasonable growth buffer, replacing a 10.9% utilization rate with a scheme where every address block serves a defined purpose. NAT/PAT at the router edge means only one public IP address is needed for all outbound internet traffic from 278 internal devices.

__Dedicated DHCP Server instead of Router or Switch__ <br>
An early design decision was to centralize DHCP on a dedicated server in VLAN 60 rather than running it on the router or the Layer 3 switch. In a healthcare environment, DHCP is a critical service because IP phones, wireless clients, and clinical devices all depend on it for connectivity. Hosting DHCP on dedicated server separates the service fro the network infrastructure, meaning a router restart or switch reload does not interrupt IP address assignment for active devices. In a production deployment this server would be paired with a secondary DHCP server for high availability, which was noted as a future consideration given Packet Tracer simulation constraints.

__Redundant Uplinks on All Backbone Connections__ <br>
Given that the original network had zero redundancy and that high availability was identified as non-negotiable business requirement, redundant uplinks were implemented on every backbone connection including router to core switch and core switch to each access switch. Spanning Tree Protocol manages the redundant paths by keeping one active and one in standby, with automatic failover if the primary link fails. This was treated as a core design requirement than an optional enhancement, reflecting the reality that in a clinical environment a network outage is an operational emergency.

## Technologies Used

| Category  | Technology |
| --------- | ---------- |
| Simulation | Cisco Packet Tracer |
| Routing Protocol | OSPF Single-Area |
| Switching | IEEE 802.1Q VLAN Trunking, Spanning Tree Protocol |
| IP Addressing | RFC1918 Private Addressing, VLSM |
| DHCP | Dedicated DHCP Server with DHCp Relay (ip helper-address) |
| Network Address Translation | NAT/PAT (PAT overload) |
| Wireless | WPA3 Enterprise 802.1X with RADIUS, Captive Portal |
| VoIP | Cisco Call Manager Express, DHCP Option 150 |
| Access Control | Extended Named ACL |
| Network Management | Centralized Network Controller, VLAN 70 Management Segment |
| Devices | Cisco 2811, Cisco 3650-24PS, Cisco 2960-24TT, Cisco IP Phone 7960, Access Point PT |

## Network Topology
## VLAN Design & IP Addressing
## Configuration Highlights
## Simulation Limitations
## Future Considerations
## Lessons Learned
