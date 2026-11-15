# Windows Server 2025: AD DS, DHCP, and DNS

## Server details

| Setting | Value |
|---|---|
| Computer name | DC01 |
| Fully qualified name | DC01.lab.local |
| Domain | lab.local |
| IPv4 | 192.168.10.10/24 (static) |
| Default gateway | 192.168.10.1 |
| Preferred DNS | 192.168.10.10 |
| Hypervisor | VMware |
| Network mode | Bridged to physical NIC |

## Installed roles

The documented server roles are:
- Active Directory Domain Services (AD DS)
- DHCP Server
- DNS Server
- File and Storage Services

## DHCP scopes

Four scopes were created using the Windows DHCP New Scope Wizard:

| Scope network | VLAN | Router option | DNS option | DNS suffix |
|---|---:|---|---|---|
| 192.168.10.0/24 | 20 | 192.168.10.1 | 192.168.10.10 | lab.local |
| 192.168.20.0/24 | 30 | 192.168.20.1 | 192.168.10.10 | lab.local |
| 192.168.30.0/24 | 40 | 192.168.30.1 | 192.168.10.10 | lab.local |
| 192.168.40.0/24 | 50 | 192.168.40.1 | 192.168.10.10 | lab.local |

The notes do not record the exact pool start/end addresses or exclusion ranges for all scopes. Add those details after checking the DHCP console; do not guess them.

## DHCP relay dependency

The switch forwards client DHCP requests from VLANs 30, 40, and 50 to `192.168.10.10`. The server's default gateway must be `192.168.10.1` so it can return traffic to remote client subnets.

## Operational notes

- The DHCP scope's Router (003) option must match that VLAN's SVI.
- DNS Servers (006) points to `192.168.10.10`.
- DNS Domain Name (015) is `lab.local`.
- Exclusion ranges must fall within the scope's configured address range.
- The WINS options 044 and 046 were noted as unnecessary for this lab.

## Next domain tasks

The recorded next steps are to join a client PC to `lab.local`, create organizational units (OUs), create users, and configure a Group Policy Object (GPO). These were listed as future work in the October 25 notes.
