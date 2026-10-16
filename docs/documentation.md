# Technical Documentation — Router-on-a-Stick Lab

## 1. Project Overview

This lab demonstrates inter-VLAN routing using the Router-on-a-Stick (ROAS) architecture in Cisco Packet Tracer.

The network contains one router, two switches, and seven PCs. End devices are logically segmented into VLANs, while R1 provides Layer 3 connectivity through subinterfaces configured on a single physical interface.

The implementation covers VLAN segmentation, access ports, IEEE 802.1Q trunking, native VLAN configuration, IP addressing, routing, and connectivity verification.

## 2. Objectives

The objectives of this lab are to:

- Create and organize VLANs on the switches.
- Assign end-device ports to the appropriate VLANs.
- Configure static 802.1Q trunk links.
- Control which VLANs are permitted across each trunk.
- Configure a native VLAN.
- Create router subinterfaces and assign gateway addresses.
- Enable communication between different VLANs.
- Validate the configuration using Cisco IOS commands and ICMP tests.

## 3. Topology and Physical Connections

The network consists of R1, SW1, SW2, and seven PCs.

The relevant inter-device connections are:

| Device | Interface | Connected device | Interface |
|---|---|---|---|
| R1 | `G0/1` | SW2 | `G0/1` |
| SW2 | `G0/2` | SW1 | `G0/2` |

### Access ports

| Switch | Ports | VLAN | Department |
|---|---|---:|---|
| SW1 | `Fa0/1–Fa0/2` | 10 | IT |
| SW1 | `Fa0/3–Fa0/4` | 30 | SALES |
| SW2 | `Fa0/1–Fa0/2` | 10 | IT |
| SW2 | `Fa0/3` | 20 | HR |

The remaining end-device ports are not part of the configured access-port assignments in this lab.

## 4. VLAN and IP Addressing Plan

The lab uses four logical IP networks, each with a `/26` prefix.

| VLAN | Name | Network | Usable host range | Default gateway | Broadcast |
|---:|---|---|---|---|---|
| 10 | IT | `10.0.0.0/26` | `10.0.0.1–10.0.0.62` | `10.0.0.62` | `10.0.0.63` |
| 20 | HR | `10.0.0.64/26` | `10.0.0.65–10.0.0.126` | `10.0.0.126` | `10.0.0.127` |
| 30 | SALES | `10.0.0.128/26` | `10.0.0.129–10.0.0.190` | `10.0.0.190` | `10.0.0.191` |
| 1001 | Native | `10.0.0.192/26` | `10.0.0.193–10.0.0.254` | `10.0.0.254` | `10.0.0.255` |

**Subnet mask:** `255.255.255.192`

Each `/26` subnet provides 62 usable IPv4 host addresses. The last usable address is assigned to the corresponding router subinterface.

The native VLAN is configured on the trunk links. Its IP subnet and gateway are configured on R1, although end-to-end use of that native VLAN depends on its Layer 2 VLAN membership and trunk allow-list configuration.

## 5. VLAN Configuration

### 5.1 SW1

SW1 defines VLAN 10 (IT) and VLAN 30 (SALES).

The access ports are assigned as follows:

- `Fa0/1–Fa0/2`: VLAN 10
- `Fa0/3–Fa0/4`: VLAN 30

The access ports use `switchport mode access` to operate as access interfaces. `switchport nonegotiate` disables Dynamic Trunking Protocol (DTP) negotiation on these ports.

### 5.2 SW2

SW2 defines VLAN 10 (IT), VLAN 20 (HR), and VLAN 30 (SALES).

The access ports are assigned as follows:

- `Fa0/1–Fa0/2`: VLAN 10
- `Fa0/3`: VLAN 20

VLAN 30 is also defined on SW2 because it must be carried over the trunk connecting SW1 to SW2.

## 6. Trunk Configuration

Trunk links transport traffic from multiple VLANs over a single physical connection. IEEE 802.1Q tagging identifies the VLAN associated with each Ethernet frame.

The lab uses static trunk configuration rather than dynamic negotiation.

### 6.1 SW1–SW2 trunk

The link between SW1 `G0/2` and SW2 `G0/2` permits VLANs 10 and 30.

| Setting | Value |
|---|---|
| SW1 interface | `G0/2` |
| SW2 interface | `G0/2` |
| Mode | Static trunk |
| Allowed VLANs | 10, 30 |
| Native VLAN | 1001 |
| DTP negotiation | Disabled |

VLAN 20 is not required on this trunk because the HR access port is connected directly to SW2.

### 6.2 SW2–R1 trunk

The link between SW2 `G0/1` and R1 `G0/1` carries the user VLANs toward the router.

| Setting | Value |
|---|---|
| SW2 interface | `G0/1` |
| R1 interface | `G0/1` |
| Mode | Static trunk on SW2 |
| Allowed VLANs | 10, 20, 30 |
| Native VLAN | 1001 |
| DTP negotiation | Disabled on SW2 |

The router uses 802.1Q encapsulation on its subinterfaces to process traffic for the relevant VLANs.

### Native VLAN considerations

Both switch trunks specify VLAN 1001 as their native VLAN. The router also configures subinterface `G0/1.1001` with native 802.1Q encapsulation.

However, the recorded trunk allow-lists contain only VLANs 10 and 30 on the SW1–SW2 link, and VLANs 10, 20, and 30 on the SW2–R1 link. VLAN 1001 is not explicitly included because not explicitly created on switches.

## 7. Router-on-a-Stick Configuration

### 7.1 Physical interface

R1 uses the physical interface `GigabitEthernet0/1` to connect to SW2.

The physical interface is enabled with `no shutdown`. The Layer 3 gateway addresses are assigned to subinterfaces rather than directly to the physical interface.

### 7.2 Subinterfaces

| Subinterface | 802.1Q configuration | IPv4 address | Purpose |
|---|---|---|---|
| `G0/1.10` | `encapsulation dot1q 10` | `10.0.0.62/26` | IT gateway |
| `G0/1.20` | `encapsulation dot1q 20` | `10.0.0.126/26` | HR gateway |
| `G0/1.30` | `encapsulation dot1q 30` | `10.0.0.190/26` | SALES gateway |
| `G0/1.1001` | `encapsulation dot1q 1001 native` | `10.0.0.254/26` | Native VLAN gateway |

Each subinterface associates an IP network with a VLAN. The gateway address must match the default gateway configured on the end devices belonging to that subnet.

### 7.3 Routing behavior

When a PC sends traffic to a destination outside its local subnet, it forwards the packet to its default gateway.

R1 receives the Ethernet frame on the trunk, identifies the VLAN using 802.1Q information, and processes the IP packet through the corresponding subinterface. If the destination belongs to another directly connected VLAN network, R1 routes the packet to that network and transmits it back through the appropriate subinterface.

This enables communication between VLANs without requiring a separate physical router interface for each network.

## 8. Verification Procedure

The lab includes command outputs in `tests/shows.txt` and screenshots in the `tests/` directory.

### 8.1 Verify VLAN membership

Run on SW1 and SW2:

```text
show vlan brief
```

Check that the expected VLANs exist and that access ports belong to the correct VLANs.

### 8.2 Verify trunk operation

Run on SW1 and SW2:

```text
show interfaces trunk
```

Check:

- Trunk status and 802.1Q encapsulation.
- Native VLAN configuration.
- Allowed VLANs.
- VLANs that are active and forwarding on each trunk.

### 8.3 Verify router interfaces

Run on R1:

```text
show ip interface brief
```

Confirm that the physical interface and the relevant subinterfaces are operational and have the expected IP addresses.

### 8.4 Verify the routing table

Run on R1:

```text
show ip route
```

The recorded output shows these directly connected networks:

- `10.0.0.0/26` through `G0/1.10`
- `10.0.0.64/26` through `G0/1.20`
- `10.0.0.128/26` through `G0/1.30`
- `10.0.0.192/26` through `G0/1.1001`

The corresponding local `/32` routes represent the router's own interface addresses.

### 8.5 Verify end-to-end connectivity

Use ICMP ping from a PC in one VLAN to a PC in another VLAN.

For example, test communication between an IT host and a SALES host using the destination host's actual configured IPv4 address.

Successful replies confirm that the path supports inter-VLAN communication for the tested hosts. The repository includes `tests/end-to-end-connectivity.png` and `tests/simulation-mode.png` as supporting evidence.

## 9. Troubleshooting Checklist

If inter-VLAN communication fails, verify the following:

1. The source and destination PCs have correct IPv4 addresses, subnet masks, and default gateways.
2. The access ports are assigned to the intended VLANs.
3. The required VLANs exist on the switches that need to carry them.
4. The trunk links are operational and permit the relevant VLANs.
5. The native VLAN configuration matches across the trunk endpoints.
6. R1's physical interface is enabled.
7. Each router subinterface uses the correct VLAN ID, encapsulation, and gateway address.
8. The routing table contains the expected directly connected networks.
9. ICMP tests are performed against valid destination addresses.

## 10. Key Takeaways

This lab demonstrates the relationship between Layer 2 VLAN segmentation and Layer 3 routing.

- VLANs separate broadcast domains.
- Access ports connect end devices to specific VLANs.
- 802.1Q trunks carry multiple VLANs over a physical link.
- Router subinterfaces provide default gateways for multiple VLANs.
- Explicit trunk allow-lists control which VLANs may cross each link.
- Show commands and end-to-end tests help validate the configuration and isolate faults.

## 11. Project Files

- `src/Router-On-A-Stick.pkt` — Packet Tracer simulation.
- `configs/R1-config.txt` — router subinterface configuration.
- `configs/SW1-config.txt` — SW1 VLAN, access-port, and trunk configuration.
- `configs/SW2-config.txt` — SW2 VLAN, access-port, and trunk configuration.
- `docs/topology.png` — network topology.
- `tests/shows.txt` — recorded Cisco IOS verification outputs.
- `tests/end-to-end-connectivity.png` — connectivity test evidence.
- `tests/simulation-mode.png` — simulation evidence.

## Author

**C. Victorin Vodinou**

[GitHub Profile](https://github.com/Victorin13)
[LinkedIn Profile](https://www.linkedin.com/in/victorin-vodinou/)
[Gmail Prifile](victorinvodinou@gmail.com)

Networking | Systems Administration | Network Security