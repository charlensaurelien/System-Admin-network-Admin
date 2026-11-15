# Validation and Test Results

This page separates results recorded as working from items that still needed testing.

## Results recorded as working (October 25, 2026)

| Test / observation | Recorded result |
|---|---|
| VLAN 20 client on Gi0/2 | Received `192.168.10.12`; DHCP snooping binding recorded |
| VLAN 30 client on Gi0/3 | Binding recorded for `192.168.20.11` |
| VLAN 40 client on Gi0/5 | Received address; binding recorded for `192.168.30.11` |
| VLAN 50 client on Gi0/7 | Received `192.168.40.11`, mask `255.255.255.0`, gateway `192.168.40.1`, DNS suffix `Lab.local` |
| Snooping table | Four bindings, one per VLAN 20, 30, 40, and 50 |
| DC01 to SVI ping | Replies from `192.168.20.1` and `192.168.30.1` with 0% loss |
| DHCP relay | Confirmed working end to end with scopes, trusted server port, and server gateway |
| Snooping database | Successfully written to flash |
| Guest DAI stale-binding issue | Deny messages stopped after clients received valid DHCP bindings |

The notes report that `192.168.40.1` was unreachable from DC01 only while the VLAN 50 SVI was down/down. Connecting a device to Gi0/7 brought the SVI up.

## Pending validation

| Item | Suggested validation |
|---|---|
| Guest ACL application | `show running-config interface vlan 50` |
| Guest DNS access | From guest client: `nslookup lab.local` |
| Guest isolation | Test access to internal subnets from a guest client |
| ACL counters | `show access-lists Guest-in` |
| Access port mode | Review Gi0/3 through Gi0/8 configuration |
| DAI on VLAN 20 | Enable only after planning and validating trusted/static hosts |
| Unused ports | Confirm Gi0/9–Gi0/12 are unused before shutdown |
| Startup configuration | Save and verify configuration persists after reload |
| Other access ports | Test clients on Gi0/1, Gi0/4, Gi0/6, and Gi0/8 as appropriate |

## Evidence to add to the repository

When collecting screenshots or command output, redact credentials, public IP addresses, serial numbers, and other sensitive details.

Useful evidence:
- `show vlan brief`
- `show ip interface brief`
- `show ip route`
- `show ip dhcp snooping`
- `show ip dhcp snooping binding`
- `show ip arp inspection`
- `show port-security`
- `show access-lists Guest-in`
- Windows DHCP scope configuration (with sensitive details removed)
- A successful client `ipconfig /all` output with identifiers redacted
