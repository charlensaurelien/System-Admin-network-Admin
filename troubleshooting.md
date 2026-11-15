# Troubleshooting Log

Issues and fixes recorded during the October 25, 2026 lab session.

| # | Symptom | Cause | Fix / lesson |
|---:|---|---|---|
| 1 | `%SW_DAI-4-DHCP_SNOOPING_DENY` messages on Gi0/7, VLAN 50, for host `192.168.10.12` | Device retained an address from the IT subnet and had no matching snooping binding in the Guest VLAN | Renewed/obtained a correct DHCP lease. DAI relies on valid snooping bindings. |
| 2 | DHCP wizard rejected an exclusion range (`192.168.10.1` to `.10`) | Exclusion range was outside the scope's start/end range | Adjusted the exclusion to fit inside the pool or removed it. |
| 3 | Only VLAN 20 clients received addresses | Scopes existed only for `192.168.10.0`; other VLANs used different subnets | Created scopes for `192.168.20.0`, `192.168.30.0`, and `192.168.40.0`. |
| 4 | Relayed DHCP requests failed | DC01 had no default gateway and could not return traffic to other subnets | Set DC01's gateway to `192.168.10.1`. |
| 5 | Only same-subnet devices could reach the server | VMware bridged adapter was not passing traffic correctly | Set a static IP on the host NIC so the bridge worked. |
| 6 | Ping to `192.168.40.1` from DC01 returned destination host unreachable | VLAN 50 SVI was down/down because no active port was in VLAN 50 | Connected a device to Gi0/7, bringing the SVI up. |
| 7 | DHCP snooping binding and filter commands failed | Incorrect command and pipe syntax | Used `show ip dhcp snooping binding` and valid `include` filtering. |
| 8 | Saving configuration failed | Typo in command (`rinning-config`) | Correct command: `copy running-config startup-config`. |

## General lessons

- DHCP relay requires a matching scope for the client subnet.
- The DHCP server needs a valid route back to remote client subnets.
- A VLAN SVI may remain down/down until the VLAN has an active Layer 2 port.
- DAI depends on valid DHCP snooping bindings unless another supported validation method is configured.
- DHCP exclusions must be within the scope's configured address range.
- Verify exact IOS command syntax on the device before relying on copied examples.
