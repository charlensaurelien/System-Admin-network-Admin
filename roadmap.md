# Project Roadmap

This roadmap reflects open items from the October 25, 2026 lab notes. Update the status as each task is completed and tested.

## Priority 1 — Verify and secure the existing setup

- [ ] Save the switch configuration with `copy running-config startup-config`.
- [ ] Confirm the `Guest-in` ACL is applied inbound on VLAN 50.
- [ ] Test guest DNS resolution and confirm guest access to internal subnets is blocked as intended.
- [ ] Review ACL hit counters with `show access-lists Guest-in`.
- [ ] Verify `switchport mode access` on Gi0/3 through Gi0/8.
- [ ] Test additional access ports: Gi0/1, Gi0/4, Gi0/6, and Gi0/8, accounting for the server connection on Gi0/1.
- [ ] Review DHCP pools and exclusions for static devices.

## Priority 2 — Switch hardening

- [ ] Confirm Gi0/9 through Gi0/12 are unused, then shut them down.
- [ ] Consider spanning-tree PortFast and BPDU Guard on suitable endpoint-facing access ports.
- [ ] Plan and enable DAI on VLAN 20 after accounting for the server and any static-IP devices.
- [ ] Review port-security violation behavior and aging settings; the source notes do not specify them.

## Priority 3 — Addressing and management

- [ ] Plan a maintenance window to renumber subnets so VLAN IDs and subnet numbers align.
- [ ] Decide whether to implement VLAN 99 for management.
- [ ] If remote management is needed, configure a management SVI and dedicated admin port.
- [ ] Configure SSH securely if remote switch administration is introduced.
- [ ] Use an unused native VLAN (for example VLAN 999) only when a trunk is added; the documented current design does not require a trunk native VLAN.

## Priority 4 — Active Directory practice

- [ ] Join a client PC to `lab.local`.
- [ ] Create OUs.
- [ ] Create test users and groups.
- [ ] Create and link a test GPO.
- [ ] Verify DNS and domain authentication from a client.

## Evidence and documentation

- [ ] Add sanitized switch command output.
- [ ] Add a current topology diagram.
- [ ] Record actual DHCP pool ranges and exclusions.
- [ ] Record test date, device, command, expected result, and actual result for each validation.
