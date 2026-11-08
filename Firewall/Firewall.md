# OPNsense Firewall

## 1. Device Overview

| Parameter | Value |
|---|---|
| Device | OPNsense Firewall |
| Role | Perimeter firewall |
| Virtualization | VMware Workstation |
| LAN interface | `em0` - 10.10.250.1/30 |
| WAN interface | `em1` - DHCP (no address in the isolated lab) |
| Management interface | `em2` - 192.168.250.2/24 |
| Connected router | R2 |

## 2. Network Interfaces

The firewall has three assigned network interfaces.

| Interface | Identifier | IP address | Purpose |
|---|---|---|---|
| LAN | `em0` | 10.10.250.1/30 | Connection to R2 |
| WAN | `em1` | DHCP | External network connectivity |
| OPT1 | `em2` | 192.168.250.2/24 | Web GUI management |

### LAN Connection

The LAN interface connects OPNsense directly to router R2.

| Device | Interface | IP address |
|---|---|---|
| OPNsense | LAN (`em0`) | 10.10.250.1/30 |
| R2 | `g0/1` | 10.10.250.2/30 |

R2 uses a static default route (0.0.0.0/0 via 10.10.250.1) pointing to OPNsense and distributes it to the other routers through OSPF.

### WAN Connection

The WAN interface is set to obtain an IPv4 address through DHCP. The lab is isolated, so the interface does not receive an address. As a result, NAT and internet access through the firewall are configured but cannot be tested.

### Management Interface

The OPT1 interface is used only to reach the web GUI from my Windows PC.

| Parameter | Value |
|---|---|
| Interface | `em2` |
| IP address | 192.168.250.2/24 |
| Access method | HTTPS |

This network is separate from the lab address space (10.10.0.0/16), so management access does not interfere with the simulated traffic.

## 3. Network Architecture

OPNsense operates as the perimeter firewall of the laboratory.

Internal routing is handled by the Cisco routers. R2 is the router connected directly to OPNsense and forwards traffic towards the firewall using its default route.

The firewall is not used for inter-VLAN routing or internal network segmentation.
Its only responsibility is controlling traffic between the internal network and external networks.

## 4. Firewall Aliases

Aliases group hosts and networks to keep the rules readable.

The following aliases are configured:

| Alias | Type | Value | Purpose |
|---|---|---|---|
| `fedora` | Host | 10.10.20.50 | Fedora server |
| `Interna` | Network | 10.10.0.0/16 | All internal lab networks |
| `Management` | Network | 10.10.99.0/24 | Management VLAN 99 |
| `Ospiti` | Network | 10.10.30.0/24 | Guest VLAN 30 |
| `r2` | Host | 10.10.250.2 | Router R2 |
| `Servers` | Network | 10.10.20.0/24 | Servers VLAN 20 |
| `Utenti` | Network | 10.10.10.0/24 | Users VLAN 10 |

`Interna` is a summary network: the lab has no segment with the address `10.10.0.0`. The alias only matches every address from `10.10.0.0` to `10.10.255.255`.

## 5. Firewall Rules

Firewall rules are configured on the LAN and OPT1 interfaces.

### OPT1 rules

| # | Action | Source | Destination | Protocol | Port | Description |
|:---:|---|---|---|---|---|---|
| 1 | Pass | 192.168.250.1 | This Firewall | TCP | HTTPS | GUI access from the administrator PC |

### LAN rules (in evaluation order)

| # | Action | Source | Destination | Protocol | Port | Description |
|:---:|---|---|---|---|---|---|
| 1 | Pass | `Management` | This Firewall | TCP | HTTPS | GUI access from the management network |
| 2 | Pass | `r2` | This Firewall | ICMP | any | Ping from R2 |
| 3 | Block | any | This Firewall | any | any | Block access to the firewall itself |
| 4 | Pass | `fedora` | any | any | any | Fedora to internet |
| 5 | Pass | `Ospiti` | any | any | any | Guest network: internet only |
| 6 | Pass | `Management`, `Servers`, `Utenti` | any | any | any | Internal networks to internet |
| 7 | Pass | `Ospiti` | `fedora` | TCP/UDP | DNS | Guest DNS to Fedora |
| 8 | Pass | `Ospiti` | `fedora` | UDP | 67 | Guest DHCP to Fedora |
| 9 | Block | `Ospiti` | `Interna` | any | any | Guest access to internal networks blocked |
| 10 | Block | any | any | any | any | Default block |

### Effective rules and rules that never match

- **Rules 1 to 3** control access to the firewall GUI and are effective: only the management network, and R2 for ping, can reach the firewall from the LAN side.
- **Rules 4 to 6** are effective for traffic towards the internet, which is the firewall's role.
- **Rules 7 to 9 never match.** Rule 5 passes all traffic from the guest network and is evaluated before them. In addition, traffic between guests and the Fedora server or other internal networks is routed by R1 and R3 and does not cross the firewall. The restrictions on guests are enforced by the router ACLs `OSPITI` (see the router configuration notes).

The rules are kept as a documented configuration. They show the policy intended for the original and one alternative design (see section 8) and would take effect if inter-VLAN traffic were moved through the firewall (in that scenario, rules order need to change).

## 7. Management

The web interface is reachable through the OPT1 interface.

| Parameter | Value |
|---|---|
| Management IP | 192.168.250.2 |
| Management interface | `em2` |
| Access method | HTTPS |

### Routing

OPNsense has one static route for the internal networks:

| Destination | Gateway | Purpose |
|---|---|---|
| `10.10.0.0/16` | R2 (10.10.250.2) | Return traffic to the lab VLANs and syslog to the Fedora server |

This is not the default gateway: the default route stays on the WAN.

## 8. Logging

The firewall sends its logs to the Fedora server so that they are stored together with the logs of routers and switches.

| Parameter | Value |
|---|---|
| Transport | UDP |
| Destination | 10.10.20.50 (Fedora server) |
| Port | 514 |

On Fedora, rsyslog stores the messages that arrive from `10.10.250.1` (the firewall LAN address) in `/var/log/remote/OPNsense.log`.

## 8. Configuration History and Design Considerations

The current ruleset reflects an earlier design in which OPNsense was meant to be a centralized firewall placed inside the internal network, with restrictions between VLANs.

The architecture was later changed: OPNsense was given a perimeter-only role, while routing and inter-VLAN filtering remained on the Cisco routers.

Most of the aliases and rules from the original design are still present. They do not match the actual traffic flow of the current topology and are therefore ineffective or redundant.
