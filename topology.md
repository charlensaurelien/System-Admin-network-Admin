# Lab Topology (Text Diagram)

```text
                    VMware host / physical NIC
                              |
                       Bridged VM NIC
                              |
                    Windows Server 2025 VM
                         DC01.lab.local
                    192.168.10.10/24
                   AD DS / DHCP / DNS
                              |
                         Gi0/1 (trusted)
                              |
                  +------------------------+
                  | Cisco Catalyst 3560    |
                  | SW1 - Layer 3 switch   |
                  | Inter-VLAN routing     |
                  +------------------------+
                    |       |       |       |
                 VLAN 20 VLAN 30 VLAN 40 VLAN 50
                  IT      HR     Sales   Guest
                    |       |       |       |
                  Gi0/2  Gi0/3-4 Gi0/5-6 Gi0/7-8
                    |       |       |       |
                 Client  Clients  Clients  Clients
```

## Gateway and subnet mapping

- VLAN 20 — `192.168.10.1/24`
- VLAN 30 — `192.168.20.1/24`
- VLAN 40 — `192.168.30.1/24`
- VLAN 50 — `192.168.40.1/24`

The server is connected in VLAN 20. The switch relays DHCP broadcasts from VLANs 30, 40, and 50 to `192.168.10.10`.

**Diagram limitation:** This is a logical diagram based on the October 25, 2026 notes. It does not represent every physical cable, the VMware host's own IP configuration, or an internet/WAN connection.
