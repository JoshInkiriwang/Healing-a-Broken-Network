# Healing a Broken Network
### Enterprise Hospital Infrastructure Redesign for Harshborough Medical Center

![Cisco Packet Tracer](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![OSPF](https://img.shields.io/badge/Routing-OSPF-green?style=for-the-badge)
![VoIP](https://img.shields.io/badge/VoIP-Cisco%20CME-orange?style=for-the-badge)
![VLANs](https://img.shields.io/badge/Segmentation-7%20VLANs-blue?style=for-the-badge)
![NAT](https://img.shields.io/badge/NAT-PAT%20Overload-red?style=for-the-badge)

---

## Overview

This project is a network modernization case study for Harshborough Hospital, a 10-floor medical facility with 263 clients and 15 servers running on a critically flawed flat topology. Every floor connected directly to a single router with no segmentation, no redundancy, and wasteful public IP addressing at 10.9% utilization. In a clinical environment where network downtime affects patient care, that architecture was not acceptable.

The redesign replaces it with a two-tier hierarchical network covering VLAN segmentation, OSPF routing, DHCP relay, NAT/PAT, VoIP via Cisco CME, centralized WLAN management, and access control, all simulated and verified in Cisco Packet Tracer.

---

## The Problem

The existing network had three compounding issues that would only get worse as the hospital expanded its digital services.

The first was a single point of failure at every level. One router handled all inter-floor traffic, server access, and internet requests simultaneously. One link failure anywhere in the topology meant an entire floor lost access to electronic health records and patient systems.

The second was addressing waste. Each floor held a full public Class C block for 254 addresses against actual device counts ranging from 14 to 43 per floor, totaling 2,540 allocated addresses for only 278 actual devices. The hospital was paying ISP costs for address blocks it did not need.

The third was the absence of segmentation. A flat topology with no VLAN boundaries meant clinical devices, staff workstations, IP phones, wireless clients, and servers all shared the same broadcast domain with no access control between them.

---

## Proposed Architecture

A Cisco 2811 router sits at the WAN edge and handles only internet connectivity and NAT/PAT. Below it, a Cisco 3650-24PS Layer 3 switch handles all inter-VLAN routing via Switch Virtual Interfaces and runs single-area OSPF with the router for dynamic route exchange. Three access switches serve dedicated segments: one for the server cluster, one for floors 1 to 5, and one for floors 6 to 10 including inpatient WLAN.

All backbone connections use redundant gigabit uplinks managed by Spanning Tree Protocol, with automatic failover if a primary link fails. The router stays entirely out of internal routing decisions.

Full topology diagrams for both the before and after states are in [`/topology`](./topology).

---

## Key Design Decisions

**Keeping OSPF instead of replacing it**

The original case brief suggested moving away from OSPF because it felt hard to manage. After working through the actual topology, the conclusion was that OSPF was not the problem; the flat topology was. Every routing decision was forced through one device with no hierarchy, which made OSPF appear more complex than it is. The redesign runs single-area OSPF only between the router and the core Layer 3 switch. OSPF became simpler not by replacing it, but by giving it a better environment to operate in.

**Function-based VLAN segmentation instead of floor-based**

Floor-based segmentation seems logical until you consider that clinical staff move between floors constantly. A doctor on floor 3 in the morning and floor 7 in the afternoon should not end up on a different subnet with different access rights because of physical location. Function-based VLANs mean a staff member always lands on the same segment with the same ACL enforcement regardless of which floor they connect from, which also makes security policy significantly cleaner to maintain.

**RFC1918 private addressing with VLSM**

Replacing ten rented public Class C blocks with a single 172.16.0.0/16 private block removes ISP dependency for internal addressing entirely. VLSM sizes each subnet to actual host count with room to grow. NAT/PAT at the router edge means only one public IP is needed for all 278 devices.

**Dedicated DHCP server**

DHCP was moved from the router to a dedicated server in VLAN 60. In a hospital, IP phones, wireless clients, and clinical devices all depend on DHCP. If DHCP runs on the router and the router reloads, every one of those devices loses connectivity simultaneously. A dedicated server decouples the service from the infrastructure so a device restart does not interrupt active clients.

**Redundant uplinks as a core requirement**

Zero redundancy was identified as a non-negotiable failure of the original design. Redundant gigabit uplinks were implemented on every backbone connection, from router to core switch and from core switch to each access switch. STP manages failover automatically. This was treated as a baseline requirement rather than an optional enhancement, because in a clinical environment a network outage is an operational emergency.

---

## VLAN Design and IP Addressing

The network migrates from ten public Class C blocks to a single 172.16.0.0/16 private block segmented into seven function-based VLANs using VLSM.

| VLAN | Name | Subnet | Capacity | Purpose |
|---|---|---|---|---|
| 10 | Staff Wired | 172.16.10.0/24 | 254 hosts | Administrative and operational staff workstations |
| 20 | Inpatient | 172.16.20.0/25 | 126 hosts | Nurse stations and clinical terminals, floors 6-10 |
| 30 | Staff WLAN | 172.16.30.0/24 | 254 hosts | Wireless access for doctors and nurses with floor roaming |
| 40 | Guest WLAN | 172.16.40.0/24 | 254 hosts | Internet-only access for patients and visitors |
| 50 | VoIP | 172.16.50.0/25 | 126 hosts | Dedicated IP phone segment |
| 60 | Servers | 172.16.60.0/25 | 126 hosts | Data center servers and dedicated DHCP server |
| 70 | Management | 172.16.70.0/26 | 62 hosts | Network device management interfaces |

---

## Verification and Testing

All tests were conducted in Cisco Packet Tracer using ping, automatic DHCP assignment, and VoIP call simulation.

| Test | Source | Destination | Method | Result |
|---|---|---|---|---|
| Guest DHCP | VLAN 40 Guest | DHCP Server (172.16.60.2) | Auto IP assignment | Success |
| Staff WLAN DHCP | VLAN 30 AP | DHCP Server (172.16.60.2) | Auto IP assignment | Success |
| Guest internet access | VLAN 40 Guest | Internet (200.100.1.2) | Ping | Success |
| Guest internal isolation | VLAN 40 Guest | VLAN 10/20/30/50/60/70 | Ping | Denied |
| Staff wired to servers | VLAN 10 Staff | VLAN 60 Servers | Ping | Success |
| Inpatient to servers | VLAN 20 Inpatient | VLAN 60 Servers | Ping | Success |
| Staff WLAN to servers | VLAN 30 Staff WLAN | VLAN 60 Servers | Ping | Success |
| Guest to servers | VLAN 40 Guest | VLAN 60 Servers | Ping | Denied |
| VoIP to servers | VLAN 50 VoIP | VLAN 60 Servers | Ping | Denied |
| Staff to management | VLAN 10 Staff | VLAN 70 Management | Ping | Success |
| Guest to management | VLAN 40 Guest | VLAN 70 Management | Ping | Denied |
| OSPF neighbor | Router Core | Layer 3 Switch | show ip ospf neighbor | Established |
| VoIP registration | IP Phone 101, 102 | CME (Router Core) | Phone registration | Registered |
| VoIP call | Extension 101 | Extension 102 | Call simulation | Connected |
| Internal to internet | All VLANs | Internet via PAT | Ping | Success |

---

## Debugging Notes

**NAT/PAT bound to wrong interface**

Guest clients were receiving IP addresses via DHCP and the SVI showed green status on all links, but internet access from internal VLANs was failing silently. Tracing the path revealed that NAT was configured on FastEthernet while the WAN cable was physically connected to GigabitEthernet. Because Packet Tracer link status shows active regardless of whether NAT is applied to the correct interface, the error was not immediately visible. The fix was straightforward once the interface mismatch was identified: NAT must be applied to the specific physical interface where the cable is connected, not just any routable interface. Green link status confirms Layer 1 connectivity, not correct Layer 3 policy binding.

**VoIP registration failure on Layer 3 switch topology**

Most CME tutorials show IP phones connecting directly to the router. In this topology, phones connect through a Layer 3 switch with SVIs, which introduces two issues not covered in standard guides.

The first issue was that the phone was not receiving a DHCP address despite the helper-address being configured on the SVI. The cause was switch port mode: the port connecting the IP phone must be configured as `switchport mode voice` rather than `switchport mode access`. Without voice mode, the switch does not tag VoIP traffic to the correct VLAN and DHCP discovery never reaches the relay.

The second issue was CME source address reachability. Because the router and switch are connected via a routed link rather than directly, the CME `ip source-address` must point to a loopback interface on the router to ensure phones always have a stable registration target regardless of which physical link is active. Once both issues were resolved, both extensions registered and calls between 101 and 102 connected successfully.

**Spanning Tree priority adjustment for redundant uplinks**

With redundant uplinks between the core switch and access switches, STP needed explicit priority configuration to ensure predictable path selection. The core switch was set as root bridge for all user VLANs using `spanning-tree vlan 20,30,40,50,70 priority 24576`. Without this, STP root election is non-deterministic and traffic may take suboptimal paths across the redundant links.

---

## Simulation Limitations

**Physical media.** Backbone connections use copper gigabit because Packet Tracer access switches do not have fiber ports. In production, backbone links would use 10GbE fiber while endpoint connections would remain copper gigabit. This does not affect logical configuration or protocol behavior.

**Network Controller.** The PT-Controller represents a placeholder for centralized WLAN management. It confirms reachability from VLAN 70 but does not simulate AP provisioning, SSID push, or CAPWAP tunnel management. A production deployment would use Cisco Catalyst Center or WLC 9800.

**DHCP high availability.** Packet Tracer does not support DHCP failover protocol. A production healthcare network would run a primary and secondary DHCP server pair with lease synchronization. The architecture accommodates a secondary server without topology changes.

**CME scale.** Two extensions are configured as representative samples. Production CME for 278 devices would require a full dial plan, hunt groups, call transfer, and voicemail integration beyond what this simulation scope covers.

---

## Future Considerations

**WLAN extension to floors 1 to 5.** The current design deploys wireless only on floors 6 to 10 per the case specification. Floors 1 to 5 have operational mobility needs for administrative staff, and guest WLAN is relevant in lobby areas and outpatient clinics on lower floors. Because all access switches already use PoE-capable gigabit uplinks, extending wireless requires only access point addition and SSID configuration with no infrastructure changes.

**DMZ for public-facing services.** Healthcare organizations increasingly operate patient portals and appointment booking systems that require internet accessibility. Exposing these from the internal server VLAN is a significant risk. A dedicated DMZ interface on the router with strict ACL separation would allow public services to operate without compromising internal clinical data.

---

## Lessons Learned

**Diagnose before proposing.** The case brief suggested replacing OSPF. Working through the actual topology revealed that OSPF was functioning correctly and the bottleneck was architectural. Replacing the protocol without fixing the topology would have produced a different-looking network with the same fundamental problem.

**ACLs interact with everything running on the same interface.** The GUEST_POLICY ACL initially broke DHCP for guest clients because DHCP discovery uses UDP ports 67 and 68, and the deny rules were evaluated before DHCP traffic could reach the relay. ACL rules must explicitly permit service-level traffic before broad deny statements, and this ordering is only obvious after breaking something and tracing why.

**Green link status is not policy confirmation.** NAT bound to the wrong interface produced no error: the link was up, routing was working, and DHCP was issuing addresses. The failure only appeared at the internet boundary. Systematic path tracing from the client outward, rather than from the failure inward, was what isolated the interface mismatch.

**Simulation creates a false sense of completeness.** Packet Tracer validates logical configuration and protocol behavior, but real deployments involve physical cable management, hardware compatibility, change management procedures, rollback planning, and the operational impact of reconfiguring a network while clinical staff are actively using it. Documenting simulation boundaries honestly is part of treating this as an engineering proposal rather than a lab exercise.

**Proportionality is a design principle.** Single-area OSPF is less impressive than multi-area, and a two-tier hierarchy is less elaborate than a full three-tier campus design, but both are the right choices at this scale. Good design matches complexity to requirements without sacrificing future growth capacity.

---

## Technologies Used

| Category | Technology |
|---|---|
| Simulation | Cisco Packet Tracer |
| Routing | OSPF Single-Area |
| Switching | IEEE 802.1Q VLAN Trunking, Spanning Tree Protocol (PVST) |
| IP Addressing | RFC1918 Private Addressing, VLSM |
| DHCP | Dedicated Server with ip helper-address relay |
| NAT | PAT overload on WAN interface |
| Wireless | WPA3 Enterprise, 802.1X with RADIUS, Captive Portal for Guest |
| VoIP | Cisco CME, DHCP Option 150, Voice VLAN port mode |
| Access Control | Extended Named ACL |
| Management | VLAN 70 dedicated management segment |

Full device configurations are in [`/configs`](./configs): router-core, switch-core, servers-switch, fl1-5-switch, fl6-10-switch.
