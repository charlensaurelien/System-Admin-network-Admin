# Network Design

## Overview

The lab uses a Cisco Catalyst 3560 Layer 3 switch (`SW1`) to connect a Windows Server 2025 VM (`DC01`) and client devices in separate VLANs.

The server VM runs in VMware and is bridged to the physical NIC connected to switch port Gi0/1. The switch provides the default gateway for each client VLAN and relays DHCP requests to the Windows DHCP service.

## Addressing and VLAN plan

| VLAN | Department / purpose | Ports | Gateway | Subnet |
|---:|---|---|---|---|
| 20 | IT | Gi0/1 server; Gi0/2 client | 192.168.10.1 | 192.168.10.0/24 |
| 30 | HR | Gi0/3–Gi0/4 | 192.168.20.1 | 192.168.20.0/24 |
| 40 | Sales | Gi0/5–Gi0/6 | 192.168.30.1 | 192.168.30.0/24 |
| 50 | Guest | Gi0/7–Gi0/8 | 192.168.40.1 | 192.168.40.0/24 |
| 99 | Management | No ports assigned yet | Not configured | Not configured |
| 1 | Default | No ports documented | 192.168.2.1 | No scope documented |

The current subnet-to-VLAN mapping works, but the numbering is not intuitive. Renumbering VLANs 30, 40, and 50 to matching third-octet subnets is a future task, not a completed change.

## Server network settings

- Hostname: `DC01`
- FQDN: `DC01.lab.local`
- Domain: `lab.local`
- IPv4: `192.168.10.10/24` (static)
- Default gateway: `192.168.10.1`
- DNS server: `192.168.10.10`

## DHCP scope options

Each Windows DHCP scope uses:
- Option 003 — Router: the SVI address for that scope's VLAN.
- Option 006 — DNS Servers: `192.168.10.10`.
- Option 015 — DNS Domain Name: `lab.local`.

| Scope subnet | Router option (003) |
|---|---|
| 192.168.10.0/24 | 192.168.10.1 |
| 192.168.20.0/24 | 192.168.20.1 |
| 192.168.30.0/24 | 192.168.30.1 |
| 192.168.40.0/24 | 192.168.40.1 |

## DHCP relay

`ip helper-address 192.168.10.10` was configured on VLAN 30, VLAN 40, and VLAN 50. VLAN 20 shares the server's subnet, so a relay is not required for the server's local VLAN.

The Windows DHCP server needs a scope for every client subnet and a working return path. Its default gateway was set to `192.168.10.1`.

## Topology

See [../diagrams/topology.md](../diagrams/topology.md) for a text-based topology representation.
