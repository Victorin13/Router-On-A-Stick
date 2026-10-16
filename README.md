# Inter-VLAN Routing — Router-on-a-Stick

## Overview

This lab demonstrates Inter-VLAN Routing using the Router-on-a-Stick
(ROAS) architecture.

Instead of using one physical router interface per VLAN, a single
router interface is configured as a trunk connection and divided
into multiple logical subinterfaces.

## Topology

The topology consists of:

- 1 Cisco router
- 2 Cisco Catalyst switches
- 7 PCs
- 3 VLANs
- 1 trunk link between the router and switch

## VLANs

| VLAN | Department | Network | Prefix |
|------|------------|---------|--------|
| 10 | IT | 10.0.0.0 | /26 |
| 20 | HR | 10.0.0.64 | /26 |
| 30 | SALES | 10.0.0.128 | /26 |

## Router Subinterfaces

| Subinterface | VLAN | Gateway |
|--------------|------|---------|
| G0/1.10 | 10 | 10.0.0.62 |
| G0/1.20 | 20 | 10.0.0.126 |
| G0/1.30 | 30 | 10.0.0.190 |

## Objectives

- Configure VLANs
- Configure access ports
- Configure a trunk link
- Configure 802.1Q encapsulation
- Create router subinterfaces
- Configure Inter-VLAN Routing
- Verify connectivity between VLANs

## Key Concepts

- VLAN
- Access Port
- Trunk
- IEEE 802.1Q
- Router-on-a-Stick
- Subinterfaces
- Inter-VLAN Routing

## Validation

Connectivity between hosts in different VLANs was successfully
verified using ICMP ping.

## Repository Structure

```text
.
├── configs/
│   └── Device configuration files (.txt)
├── docs/
│   ├── documentation.md
│   └── topology.png
├── src/
│   └── Router-On-A-Stick.pkt
├── tests/
│   ├── simulation-mode.png
│   ├── end-to-end-connectivity.png
│   ├── shows.txt
│   └── Additional verification screenshots
├── .gitignore
├── LICENSE
└── README.md
```
## Author

**C. Victorin Vodinou**

_Networking_ | _Systems Administration_ | _Network Security_

This project is part of my hands-on networking lab portfolio.