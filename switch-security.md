# Cisco 3560 Switching and Security

## Layer 3 switching

The switch has `ip routing` enabled and SVIs for VLANs 20, 30, 40, 50, and VLAN 1. The SVIs provide default gateways for the documented subnets.

## DHCP relay

The following helper address was configured on the remote client VLAN interfaces:

```text
ip helper-address 192.168.10.10
```

Documented on VLAN 30, VLAN 40, and VLAN 50.

## DHCP snooping

Documented controls:
- DHCP snooping enabled globally.
- DHCP snooping enabled for VLANs 20, 30, 40, and 50.
- Gi0/1 (server-facing port) trusted.
- DHCP rate limit of 15 on access ports.
- Snooping database saved to `flash:snoop.db`.
- `no ip dhcp snooping information option` configured so relayed requests are accepted by the Windows DHCP server.

DHCP snooping builds a binding table containing client MAC address, IP address, VLAN, switch port, and lease information. It helps block unauthorized DHCP server responses.

## Dynamic ARP Inspection (DAI)

Documented state:
- DAI enabled for VLANs 30, 40, and 50.
- Gi0/1 trusted.
- VLAN 20 was **not yet enabled** in the recorded configuration.

DAI checks ARP traffic against DHCP snooping bindings. Devices using static IP addresses may need an ARP ACL or a carefully justified trusted-port design.

## Port security

Sticky MAC learning and a maximum of two MAC addresses were documented on ports Gi0/1 through Gi0/8.

The notes do not specify a violation action or aging configuration. Confirm the live switch configuration before documenting those settings.

## Guest VLAN ACL

An extended ACL named `Guest-in` was written for inbound application on VLAN 50. Its intended policy is to allow DHCP and DNS to the domain controller, block access to selected internal subnets, and permit other IP traffic.

**Important:** The notes say the ACL still needed confirmation that it was applied and tested. Do not describe guest isolation as verified until testing is complete. See `../configs/guest-acl-reference.txt`.

## Remaining hardening tasks

- Verify `switchport mode access` on Gi0/3 through Gi0/8.
- Shut down unused ports Gi0/9 through Gi0/12.
- Consider PortFast and BPDU Guard on appropriate edge/access ports.
- Enable and validate DAI on VLAN 20.
- Validate the Guest-in ACL and its DHCP behavior.
- Save the switch configuration to startup configuration.
