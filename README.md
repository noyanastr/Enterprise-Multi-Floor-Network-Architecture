# Enterprise Multi-Floor Network Architecture (Cisco Packet Tracer)

A scalable, segmented 3-floor enterprise network architecture designed and simulated in Cisco Packet Tracer. Built around a hierarchical star topology, this project supports 90 client workstations across 6 corporate departments, dedicated server infrastructure, and centralized IT management.

---

## Network Topology & Hardware

The network uses a **Hierarchical Star Topology** to ensure scalability, fault isolation, and centralized traffic control.

| Layer | Device Name | Model | Quantity | Role |
| :--- | :--- | :--- | :---: | :--- |
| **Gateway / Edge** | `Temsili-FortiGate` | Cisco 2911 Router | 1 | Inter-VLAN Routing (Router-on-a-Stick) & Centralized DHCP Server |
| **Core / Distribution** | `SW-Core` | Cisco Catalyst 3560-24PS | 1 | Central backbone aggregation, Server & IT Admin connectivity |
| **Access Layer** | `SW-Kat1-A` – `SW-Kat3-B` | Cisco Catalyst 2960-24TT | 6 | Floor/Department access switches (2 per floor, 15 PCs per switch) |
| **Server Infrastructure** | `Sunucu-AD-DNS`, `Sunucu-Dosya` | Server-PT | 2 | Internal Active Directory/DNS and File Services |
| **IT Management** | `IT-Admin-1` – `IT-Admin-3` | PC-PT | 3 | Dedicated network administration workstations |
| **End Devices** | Department Workstations | PC-PT | 90 | 6 departments x 15 workstations |

### Cabling Standards
- **End Devices to Switches:** Copper Straight-Through
- **Core Switch (`SW-Core`) to Router (`Temsili-FortiGate`):** Copper Straight-Through
- **Access Switches (`Gi0/1`) to Core Switch (`Fa0/18–23`):** Copper Cross-Over

---

## VLAN Segmentation & IP Addressing Plan

The `10.10.0.0/16` address space is segmented into eight `/24` subnets to minimize broadcast domains and isolate departmental traffic.

| VLAN ID | VLAN Name | Department / Role | Location / Switch | Subnet | Default Gateway | IP Allocation Method & Range |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| **10** | `PAZARLAMA` | Marketing | 1st Floor Left (`SW-Kat1-A`) | `10.10.10.0/24` | `10.10.10.1` | **DHCP:** `10.10.10.21 – .254` |
| **20** | `DESTEK` | Support | 1st Floor Right (`SW-Kat1-B`) | `10.10.20.0/24` | `10.10.20.1` | **DHCP:** `10.10.20.21 – .254` |
| **30** | `AR_GE` | R&D | 2nd Floor Left (`SW-Kat2-A`) | `10.10.30.0/24` | `10.10.30.1` | **DHCP:** `10.10.30.21 – .254` |
| **40** | `INSAN_KAYNAKLARI` | Human Resources | 2nd Floor Right (`SW-Kat2-B`) | `10.10.40.0/24` | `10.10.40.1` | **DHCP:** `10.10.40.21 – .254` |
| **50** | `FINANS` | Finance | 3rd Floor Left (`SW-Kat3-A`) | `10.10.50.0/24` | `10.10.50.1` | **DHCP:** `10.10.50.21 – .254` |
| **60** | `UST_YONETIM` | Executive Management | 3rd Floor Right (`SW-Kat3-B`) | `10.10.60.0/24` | `10.10.60.1` | **DHCP:** `10.10.60.21 – .254` |
| **99** | `IT_MGMT` | IT Management | All Switches + 3 Admin PCs | `10.10.99.0/24` | `10.10.99.1` | **Static:** Switches (`.2`, `.11–.16`), Admins (`.10`, `.20`, `.30`) |
| **100** | `SERVERS` | Server Room | Connected to `SW-Core` | `10.10.100.0/24` | `10.10.100.1` | **Static:** AD/DNS (`.10`), File Server (`.11`) |

*Note: The first 20 IP addresses (`10.10.X.1 – 10.10.X.20`) in all departmental DHCP pools are excluded via `ip dhcp excluded-address` for gateways and static reservations.*

---

## Physical Port Mapping

### Core Switch (`SW-Core` — Cisco 3560-24PS)
- `Fa0/1 – Fa0/2`: Server Room (`Sunucu-AD-DNS`, `Sunucu-Dosya`) — `Access VLAN 100`, `PortFast`
- `Fa0/11 – Fa0/13`: IT Admin Workstations — `Access VLAN 99`, `PortFast`
- `Fa0/18 – Fa0/23`: Uplinks to 6 Access Switches — `802.1Q Trunk` (Allowed VLANs: `10,20,30,40,50,60,99,100`)
- `Fa0/24`: Uplink to `Temsili-FortiGate` (`Gi0/0`) — `802.1Q Trunk`

### Access Switches (`SW-Kat1-A` through `SW-Kat3-B` — Cisco 2960-24TT)
- `Gi0/1`: Uplink to `SW-Core` — `802.1Q Trunk` (Allowed VLANs: respective Department VLAN + `VLAN 99`)
- `Fa0/1 – Fa0/24`: End-user workstation ports — `Access` mode locked to department VLAN with `spanning-tree portfast`

---

## Key Technical Implementations

1. **802.1Q Trunking & VLAN Pruning:** Trunk links strictly allow only required VLANs on each access switch uplink (`Department VLAN` + `VLAN 99`) to reduce unnecessary broadcast propagation.
2. **Spanning Tree Optimization:** `spanning-tree portfast` is enabled on all client, admin, and server access ports to bypass the 30-second STP listening/learning delay.
3. **Out-of-Band Style Management (`VLAN 99`):** Every Layer 2 and Layer 3 switch is configured with a dedicated SVI (`interface vlan 99`) and default gateway (`10.10.99.1`) for centralized administration.
4. **Router-on-a-Stick Inter-VLAN Routing:** Subinterfaces (`Gi0/0.10` through `Gi0/0.100`) on `Temsili-FortiGate` handle inter-subnet routing using `encapsulation dot1Q`.
5. **Centralized DHCP Distribution:** Six isolated DHCP pools automatically assign IP addresses, subnet masks, default gateways, and the internal DNS server address (`10.10.100.10`) to client workstations.
6. **CLI Best Practices:** Disabled accidental DNS lookups (`no ip domain-lookup`) across all network devices and saved running configurations to NVRAM (`write memory`).

---

## Verification & Testing

The following baseline tests were executed and verified in Cisco Packet Tracer:
- **Trunk Negotiation:** Confirmed active `dot1q` trunking across `Fa0/18–24` on `SW-Core` and `Gi0/1` on all floor switches via `show interfaces trunk`.
- **Management Plane Connectivity:** Verified ICMP reachability between IT Admin workstations, `SW-Core` (`10.10.99.2`), and all 6 access switches (`10.10.99.11 – .16`).
- **Dynamic Addressing:** Confirmed successful DHCP lease acquisition across all 6 departmental subnets (`10.10.10.21` – `10.10.60.21+`).
- **Inter-VLAN & Server Reachability:** Verified end-to-end routing between departmental VLANs and static servers (`10.10.100.10`, `10.10.100.11`).

---

## How to Use

1. Install **Cisco Packet Tracer** (version 8.0 or higher recommended).
2. Clone this repository or download the `.pkt` file.
3. Open the project file in Cisco Packet Tracer and inspect configurations via device CLI (`show running-config`, `show vlan brief`, `show ip route`).
