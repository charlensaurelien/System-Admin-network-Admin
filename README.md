# Cisco Catalyst 3560 + Windows Server 2025 Home Lab

A hands-on networking and systems administration project documenting a segmented home lab built with a Cisco Catalyst 3560 Layer 3 switch and a Windows Server 2025 virtual machine.

The project demonstrates VLAN segmentation, inter-VLAN routing, centralized DHCP and DNS, DHCP relay, DHCP snooping, Dynamic ARP Inspection (DAI), port security, and a planned guest-network ACL.

> **Documentation status:** This repository reflects the lab status recorded on October 25, 2026. Items marked **planned**, **pending**, or **not yet verified** should not be treated as completed security controls.

## Project goals

- Build multiple VLANs for separate departments and guest devices.
- Use a Windows Server 2025 domain controller as the centralized DHCP and DNS server.
- Route between VLANs using the Cisco 3560 Layer 3 switch.
- Forward DHCP requests from client VLANs to the server with `ip helper-address`.
- Reduce common Layer 2 risks with DHCP snooping, DAI, and port security.
- Document configuration decisions, troubleshooting, validation, and future improvements.

## Lab components

| Component | Details |
|---|---|
| Layer 3 switch | Cisco Catalyst 3560, hostname `SW1`, IOS 15.2 |
| Server | Windows Server 2025 VM, hostname `DC01` |
| Hypervisor | VMware; VM network adapter bridged to the physical NIC |
| Domain | `lab.local` |
| Server roles | AD DS, DHCP, DNS, File and Storage |
| Server address | `192.168.10.10/24` |
| Server default gateway | `192.168.10.1` |
| Server DNS | `192.168.10.10` |
| Switch management | Console cable using PuTTY on COM4 |
| Management VLAN | VLAN 99 exists; no ports or SVI configured in the documented state |

## Network layout

| VLAN | Name | Switch ports | SVI / default gateway | Client subnet |
|---:|---|---|---|---|
| 20 | IT | Gi0/1 (server, trusted), Gi0/2 | 192.168.10.1 | 192.168.10.0/24 |
| 30 | HR | Gi0/3, Gi0/4 | 192.168.20.1 | 192.168.20.0/24 |
| 40 | Sales | Gi0/5, Gi0/6 | 192.168.30.1 | 192.168.30.0/24 |
| 50 | Guest | Gi0/7, Gi0/8 | 192.168.40.1 | 192.168.40.0/24 |
| 99 | Management | None assigned | Not configured | None |
| 1 | Default | None documented | 192.168.2.1 | No DHCP scope documented |

**Addressing note:** The VLAN IDs and subnet numbers do not align for VLANs 30, 40, and 50. For example, VLAN 30 uses `192.168.20.0/24`. This is functional in the documented setup, but renumbering is a planned cleanup task.

## How DHCP works in this lab

1. A client sends a DHCP broadcast on its local VLAN.
2. The Layer 3 switch receives the request on the VLAN interface.
3. The `ip helper-address 192.168.10.10` setting relays the request to `DC01`.
4. Windows DHCP selects the scope for the client's subnet and returns the lease information.
5. The client receives an address, subnet mask, VLAN gateway, DNS server, and DNS suffix.

The server uses `192.168.10.10` for DNS. Each DHCP scope's Router option (003) must point to that VLAN's SVI.

## Implemented controls

| Control | Documented state |
|---|---|
| Inter-VLAN routing | `ip routing` enabled |
| DHCP relay | Configured on VLANs 30, 40, and 50 |
| DHCP snooping | Enabled globally and for VLANs 20, 30, 40, and 50 |
| Trusted DHCP port | Gi0/1 |
| DHCP rate limiting | 15 packets per second on documented access ports |
| DHCP snooping database | `flash:snoop.db` |
| Dynamic ARP Inspection | Enabled for VLANs 30, 40, and 50; Gi0/1 trusted |
| Port security | Maximum 2 MAC addresses with sticky learning on Gi0/1–Gi0/8 |
| Guest ACL | Written, but application and behavior still require verification |

## Validation recorded

- Clients in VLANs 20, 30, 40, and 50 received DHCP leases.
- The DHCP snooping binding table showed a binding for each of those VLANs.
- DHCP relay, Windows DHCP scopes, the trusted server port, and the server gateway worked together end to end.
- DAI deny messages associated with a stale address stopped after clients obtained valid DHCP bindings.

See [docs/validation.md](docs/validation.md) for the recorded results and [docs/troubleshooting.md](docs/troubleshooting.md) for issues encountered.

## Repository contents

```text
.
├── README.md
├── LICENSE
├── .gitignore
├── configs/
│   ├── switch-reference-commands.txt
│   └── guest-acl-reference.txt
├── docs/
│   ├── network-design.md
│   ├── windows-server-dhcp-dns.md
│   ├── switch-security.md
│   ├── validation.md
│   ├── troubleshooting.md
│   └── roadmap.md
└── diagrams/
    └── topology.md
```

## Important security and accuracy notes

- The command files are **reference snippets**, not a complete switch running configuration. Review interface assignments and the live configuration before applying anything.
- The guest ACL was documented as written but still needed application and testing.
- DAI was not yet enabled on VLAN 20 in the documented state.
- Access mode was still missing from some port configurations and needed verification.
- Do not expose this lab directly to the public internet. Use isolated lab networks and strong, unique credentials.
- Redact public IP addresses, passwords, private keys, serial numbers, and other sensitive information before publishing screenshots or configuration backups.

## Future improvements

See [docs/roadmap.md](docs/roadmap.md). Planned work includes validating the guest ACL, hardening unused switch ports, joining a client to the domain, creating OUs and Group Policy, and adding a management VLAN for SSH administration.

## Skills demonstrated

Cisco switching and VLANs · Layer 3 switching · Inter-VLAN routing · DHCP relay · Windows Server DHCP and DNS · Active Directory Domain Services · DHCP snooping · Dynamic ARP Inspection · Port security · ACL planning · Network troubleshooting · Technical documentation
