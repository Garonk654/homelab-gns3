# OPNsense Firewall

## 1. Device Overview

| Parameter | Value |
|---|---|
| Device | OPNsense Firewall |
| Role | Perimeter Firewall |
| Virtualization | VMware Workstation |
| LAN Interface | em0 - 10.10.250.1/30 |
| WAN Interface | em1 - DHCP |
| Management Interface | em2 - 192.168.250.2/24 |
| Connected Router | R2 |

## 2. Network Interfaces

The firewall has three assigned network interfaces.

| Interface | Identifier | IP Address | Purpose |
|---|---|---|---|
| LAN | em0 | 10.10.250.1/30 | Connection to R2 |
| WAN | em1 | DHCP | External network connectivity |
| OPT1 | em2 | 192.168.250.2/24 | Web GUI management |

### LAN Connection

The LAN interface connects OPNsense directly to router R2.

| Device | Interface | IP Address |
|---|---|---|
| OPNsense | LAN (em0) | 10.10.250.1/30 |
| R2 | G0/1 | 10.10.250.2/30 |

R2 uses a static default route pointing to OPNsense.

### WAN Connection

The WAN interface is configured to obtain an IPv4 address through DHCP.

The external connectivity configuration is still pending validation.

### Management Interface

The OPT1 interface provides access to the OPNsense web interface.

- Interface: em2
- IP address: 192.168.250.2/24
- Access method: HTTPS

## 3. Network Architecture

OPNsense operates as the perimeter firewall of the laboratory.

Internal routing is handled by the Cisco routers. R2 is the router connected directly to OPNsense and forwards traffic towards the firewall using its default route.

The firewall is not used for inter-VLAN routing or internal network segmentation.

Its primary responsibility is controlling traffic between the internal network and external networks.

## 4. Firewall Aliases

Aliases are used to simplify firewall rule configuration by grouping hosts and networks.

The following aliases are configured:

| Alias | Type | Value | Description |
|---|---|---|---|
| fedora | Host | 10.10.20.50 | Fedora Server |
| Interna | Network | 10.10.0.0/16 | Internal laboratory networks |
| Management | Network | 10.10.99.0/24 | Management VLAN 99 |
| Ospiti | Network | 10.10.30.0/24 | Guest VLAN 30 |
| r2 | Host | 10.10.250.2 | Router R2 |
| Servers | Network | 10.10.20.0/24 | Servers VLAN 20 |
| Utenti | Network | 10.10.10.0/24 | Users VLAN 10 |

## 5. Firewall Rules

Firewall rules are configured on the LAN and OPT1 interfaces.

### OPT1 Rules

| Source | Destination | Protocol | Port | Description |
|---|---|---|---|---|
| 192.168.250.1 | This Firewall | TCP | HTTPS | Personal PC GUI access |

### LAN Rules

The following rules are visible in the current configuration:

| Source | Destination | Protocol | Port | Description |
|---|---|---|---|---|
| Management | This Firewall | TCP | HTTPS | Management network GUI access |
| r2 | This Firewall | ICMP | Any | Ping from R2 |
| Any | This Firewall | Any | Any | Default GUI access block |
| fedora | Any | Any | Any | Fedora to Internet |
| Ospiti | Any | Any | Any | Guest network Internet access |
| Management, Servers, Utenti | Any | Any | Any | Internal networks to Internet |
| Ospiti | fedora | TCP/UDP | DNS | Guest DNS access to Fedora |
| Ospiti | fedora | UDP | 67 | Guest DHCP access to Fedora |
| Ospiti | Interna | Any | Any | Block guest access to internal networks |
| Any | Any | Any | Any | Default block |

### Rule Evaluation

OPNsense evaluates firewall rules in order, from top to bottom.

The current configuration should be considered as a documented configuration snapshot, not proof that every intended traffic policy is operational.

## 6. Firewall Scope and Limitations

The firewall is deployed as a perimeter security device.

The Cisco routers handle internal routing, including communication between VLANs.

As a result:

- Inter-VLAN traffic is routed by R1 and R3.
- Internal traffic does not normally pass through OPNsense.
- OPNsense primarily handles traffic forwarded by R2 towards external networks.
- Internal VLAN access restrictions must be implemented on the routing infrastructure if inter-VLAN filtering is required.

The current firewall rules include some internal network policies that are not enforced by OPNsense when the traffic does not traverse the firewall.

## 7. Management

The OPNsense web interface is accessible through the OPT1 interface.

| Parameter | Value |
|---|---|
| Management IP | 192.168.250.2 |
| Management Interface | em2 |
| Access Method | HTTPS |

## 8. Configuration History and Design Considerations

The current firewall ruleset reflects an earlier design approach in which OPNsense was initially envisioned as a centralized firewall positioned within the internal network architecture.

Under that original design, OPNsense was expected to handle a broader range of traffic policies, including restrictions and permissions between internal VLANs and network segments.

The laboratory architecture was subsequently changed, and OPNsense was assigned a perimeter-only role, while internal routing and inter-VLAN traffic handling remained the responsibility of the Cisco routers.

As a result, several firewall aliases and rules are still present from the original centralized firewall design.

These rules do not necessarily correspond to the actual traffic flow in the current topology and may therefore be ineffective or redundant in the present deployment.