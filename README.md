## Overview

This lab is a segmented home network built around a Windows Server 2025 domain controller (**DC01**, domain `lab.local`) and a Layer 3 Cisco Catalyst 3560 switch (**SW1**). DC01 provides DHCP and DNS to four VLANs through DHCP relay on the switch. The access ports are protected with DHCP snooping, Dynamic ARP Inspection (DAI), port security and DHCP rate limiting.

**Result:** clients in VLAN 20, 30, 40 and 50 all receive correct IP addresses from DC01, and the switch's DHCP snooping binding table shows an entry for each VLAN.

## Topology

| Item | Details |
| --- | --- |
| Switch | Cisco Catalyst 3560, hostname SW1, IOS 15.2, Layer 3 routing enabled |
| Server | Windows Server 2025 VM (DC01.lab.local) in VMware, bridged to the physical NIC on Gi0/1 |
| Server roles | AD DS, DHCP, DNS, File and Storage |
| Server IP | 192.168.10.10/24, gateway 192.168.10.1, DNS 192.168.10.10 (static) |
| Management | Console cable (PuTTY on COM4). VLAN 99 exists but has no ports yet |

### VLAN and port plan

| VLAN | Name | Ports | SVI (gateway) | DHCP scope |
| --- | --- | --- | --- | --- |
| 20 | IT | Gi0/1 (server, trusted), Gi0/2 | 192.168.10.1 | 192.168.10.0/24 |
| 30 | HR | Gi0/3, Gi0/4 | 192.168.20.1 | 192.168.20.0/24 |
| 40 | Sales | Gi0/5, Gi0/6 | 192.168.30.1 | 192.168.30.0/24 |
| 50 | Guest | Gi0/7, Gi0/8 | 192.168.40.1 | 192.168.40.0/24 |
| 99 | Management | none yet | not configured | none |
| 1 | Default | none | 192.168.2.1 | none |

![Topology diagram](screenshots/topology.png)
<img width="2605" height="1087" alt="Screenshot 2026-11-14 203143" src="https://github.com/user-attachments/assets/d08f376d-3c9d-496a-9512-0ccc8b675222" />

<img width="5712" height="3213" alt="IMG_9506 1" src="https://github.com/user-attachments/assets/6b729477-3866-47b9-bbfd-8a61b0589ede" />


## How I Built It

### Windows Server (DC01)

- Installed and configured the **DHCP** role with the New Scope Wizard.
- Created four scopes: 192.168.10.0, 192.168.20.0, 192.168.30.0 and 192.168.40.0.
- Set these options on each scope: **003 Router** (that VLAN's SVI), **006 DNS Servers** (192.168.10.10) and **015 DNS Domain Name** (Lab.local).
- Learned that exclusion ranges must sit inside the scope's address range (this fixed the "IP address range is not a subset of the overall range" error).
- Set the server's **default gateway to 192.168.10.1**.
- Set a static IP on the VMware host NIC so the bridged adapter could pass traffic.
- Noted that the WINS options (044 and 046) are unnecessary but harmless.

DHCP scopes on DC01

<img width="3213" height="5712" alt="IMG_9509" src="https://github.com/user-attachments/assets/4c06dd33-7488-4d83-924e-918ea9846de2" />
<img width="3213" height="5712" alt="IMG_9510" src="https://github.com/user-attachments/assets/11dca7cd-f94a-4af3-9ed7-e968975e6c92" />


### Cisco 3560 switch (SW1)

**Routing and DHCP relay**

- `ip routing` enabled.
- SVIs created for VLANs 20, 30, 40 and 50 (and VLAN 1).
- `ip helper-address 192.168.10.10` on Vlan30, Vlan40 and Vlan50, so DHCP broadcasts are relayed to DC01.

**DHCP snooping**

- `ip dhcp snooping` enabled globally and for VLANs 20, 30, 40 and 50.
- `no ip dhcp snooping information option`, so relayed requests are accepted by the Windows DHCP server.
- `ip dhcp snooping trust` on Gi0/1 (the server port).
- `ip dhcp snooping limit rate 15` on access ports.
- Binding database saved to flash with `ip dhcp snooping database flash:snoop.db`.

**Dynamic ARP Inspection**

- `ip arp inspection vlan 30,40,50`.
- `ip arp inspection trust` on Gi0/1.

**Port security**

- `switchport port-security maximum 2` with sticky MAC addresses on access ports Gi0/1 to Gi0/8.

![Switch running configuration]

<img width="1502" height="1797" alt="Screenshot 2026-11-15 103918" src="https://github.com/user-attachments/assets/607273f5-9d4d-46ca-8598-a4d5bf7be372" />
<img width="1477" height="1665" alt="Screenshot 2026-11-15 103701" src="https://github.com/user-attachments/assets/d6b74e49-c1ba-492d-ac29-8e431e41742d" />

### Guest isolation ACL (VLAN 50)

I wrote an extended ACL named `Guest-in` to apply inbound on the Vlan50 SVI:

```
ip access-list extended Guest-in
 permit udp any eq bootpc any eq bootps
 permit udp any host 192.168.10.10 eq domain
 permit tcp any host 192.168.10.10 eq domain
 deny ip any 192.168.10.0 0.0.0.255
 deny ip any 192.168.20.0 0.0.0.255
 deny ip any 192.168.30.0 0.0.0.255
 deny ip any 192.168.2.0 0.0.0.255
 permit ip any any
interface vlan 50
 ip access-group Guest-in in
```

It allows DHCP and DNS to DC01 only, blocks all internal subnets, and permits everything else (for future internet access).

<img width="992" height="1697" alt="Screenshot 2026-11-15 104324" src="https://github.com/user-attachments/assets/009557cf-cca5-4cee-b1fd-14a6cd5e047f" />

**Status:** applied to the Vlan50 SVI and tested (see Follow-Up Work Completed).

## Troubleshooting

| # | Symptom | Cause | Fix |
| --- | --- | --- | --- |
| 1 | Switch log flooded with `%SW_DAI-4-DHCP_SNOOPING_DENY` on Gi0/7 (VLAN 50), host 192.168.10.12 | The device kept a leftover address from the IT subnet. In the Guest VLAN it had no snooping binding, so DAI dropped its ARP requests | Understood that DAI relies on the snooping binding table. Fixed by getting devices proper DHCP leases |
| 2 | DHCP wizard error when adding an exclusion (192.168.10.1 to .10) | The exclusion range fell outside the scope's start/end range | Adjusted the exclusion to sit inside the pool, or removed it |
| 3 | Only VLAN 20 clients got an address. Ports in VLANs 30 and 50 got none | The server had a scope only for 192.168.10.0, and the SVIs for the other VLANs used different subnets | Created scopes matching each SVI subnet (20.0, 30.0, 40.0) |
| 4 | Relayed requests still failed | DC01 had a **blank default gateway**. It received the relayed requests but could not send replies back to other subnets | Set the gateway to 192.168.10.1 on DC01 |
| 5 | Only same-subnet devices could reach the server | The VMware bridged adapter was not passing traffic properly | Gave the host NIC a static IP so the bridge worked |
| 6 | `ping 192.168.40.1` from DC01 returned "Destination host unreachable" from 192.168.10.1 | The Vlan50 SVI was up/down because no active port was in VLAN 50 | Connecting a device to Gi0/7 brought the SVI up |
| 7 | `show ip dhcp snooping binding` and filter commands failed | Used the wrong command (`show ip dhcp binding`) and wrong pipe syntax | Used `show ip dhcp snooping binding` and `\| include` filters |
| 8 | `copy rinning-config` failed | Typo in the command | Corrected to `copy running-config startup-config` |


## What I Found (Confirmed Working)

- **VLAN 20 (Gi0/2):** client received 192.168.10.12, shown in the DHCP snooping binding table.
- **VLAN 30 (Gi0/3):** binding for 192.168.20.11.
- **VLAN 40 (Gi0/5):** client received an address, and the binding shows 192.168.30.11.
- **VLAN 50 (Gi0/7):** client received **192.168.40.11**, mask 255.255.255.0, **gateway 192.168.40.1**, DNS suffix Lab.local.
- **Snooping binding table** showed **4 bindings** (one each for VLAN 20, 30, 40 and 50).
- **DC01 pings:** replies from 192.168.20.1 and 192.168.30.1 with 0% loss. 192.168.40.1 was unreachable only until Vlan50 came up.
- **DHCP relay, scopes, trusted server port and the server gateway** all work together end to end.
- **Snooping database** was successfully written to flash (the NTP warning is harmless, since the switch has no time source).
- **Scope options** on the 192.168.20.0 scope were confirmed: router 192.168.20.1, DNS 192.168.10.10, domain Lab.local.
- The DAI deny messages for the earlier stale lease stopped once clients had valid bindings.

![DHCP snooping binding table]

<img width="1930" height="410" alt="Screenshot 2026-11-15 110657" src="https://github.com/user-attachments/assets/ae17ed46-adcf-43cb-9176-e37429f83980" />

## Key Concepts Learned

- **DHCP snooping** marks ports trusted or untrusted by manual configuration, blocks rogue DHCP servers and builds a binding table (MAC, IP, VLAN, port, lease).
- **DAI** checks ARP packets against that binding table. Static-IP hosts need an ARP ACL or a trusted port.
- **DHCP relay (`ip helper-address`)** lets the switch forward broadcasts to a server in another VLAN as unicast. The server needs a scope for the client's subnet and a route back.
- **Router option (003)** must be the SVI of that scope's own VLAN.
- **An SVI is up/up only when its VLAN has an active port.**
- **Native VLAN** applies only to trunks. Access ports use their access VLAN for untagged traffic.
- **Rate limit 15** protects against DHCP starvation by err-disabling a port that floods DHCP packets.
- **ACLs** are processed top to bottom, with an implicit `deny ip any any` at the end, and wildcard masks are the inverse of subnet masks.
- Active directory settings(users ,OU,GPO,services)

## Follow-Up Work Completed

After the first round of troubleshooting, I finished the open items from my notes:

- **Saved the configuration** with `copy running-config startup-config`, so the setup survives a reboot.
- **Applied and tested the Guest-in ACL** on the Vlan50 SVI. From the Gi0/7 PC I checked that `nslookup lab.local` works, that `ping 192.168.10.10` is blocked, and that `show access-lists Guest-in` shows hit counts.
- **Added `switchport mode access`** to Gi0/3 through Gi0/8, which was missing in the running config.
- **Shut down the unused ports** (Gi0/9 to Gi0/12).






