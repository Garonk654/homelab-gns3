# Enterprise Homelab Network

A multi-layer enterprise network lab designed to simulate a small corporate infrastructure. This project focuses on implementing enterprise networking technologies, network segmentation, redundancy, dynamic routing, and security mechanisms using Cisco routers and switches.

## Project Overview

This project consists of designing, implementing, and documenting a small enterprise network in a virtualized environment. It brings together routing, switching, network security, firewalling, and centralized Linux services to replicate the core components of a corporate IT infrastructure.

### Lab Environment & Tools

The lab is built using the following platforms:

- GNS3: Network topology design, emulation, and testing of Cisco routers and switches.
- VMware Workstation: Virtualization platform used to run the lab environment and virtual machines.
- Fedora Linux: Server operating system hosting centralized network services.
- OPNsense: Firewall platform used for network security, traffic filtering, and external connectivity.

## Network Features

### Switching
- VLAN segmentation
- IEEE 802.1Q trunking
- EtherChannel
- Rapid PVST+
- STP Root Primary / Secondary
- Root Guard
- PortFast and BPDU Guard

### Routing
- Inter-VLAN Routing (Router-on-a-Stick)
- OSPF Area 0
- OSPF MD5 authentication
- Default route propagation through OSPF
- HSRP gateway redundancy
- Static routing

### Network Security
- Port Security
- DHCP Snooping
- Dynamic ARP Inspection (DAI)
- Extended Access Control Lists (ACLs)
- SSHv2 secure management
- STP security features
- OSPF authentication
- Firewall traffic filtering with OPNsense
- Fail2ban for SSH brute-force protection
- Disable unnecessary services

### Network Services
- Centralized DHCP server and DHCP Relay
- Centralized Syslog using rsyslog
- Authenticated NTP
- NFS network file sharing
- DNS services
- SSH remote administration

## VLAN Architecture

| VLAN ID | Name | Subnet | Purpose |
|:---:|---|---|---|
| 10 | UTENTI | 10.10.10.0/24 | Corporate users |
| 20 | SERVERS | 10.10.20.0/24 | Servers |
| 30 | OSPITI | 10.10.30.0/24 | Guest network |
| 99 | MANAGEMENT | 10.10.99.0/24 | Network management |
| 999 | NATIVE | N/A | Native VLAN for trunk links |

## Infrastructure

### Routers
- R1
- R2
- R3

### Switches
- SW1
- SW2
- SW3

### Firewall
- OPNsense

### Servers

**Fedora Server**

Provides centralized network services:
- DHCP
- DNS
- NFS
- NAS
- Syslog
- FAIL2BAN

  
### Routing and Switching Architecture

-R1 and R3 provide inter-VLAN routing through Router-on-a-Stick.
-HSRP provides default gateway redundancy, with R1 configured as the primary router and R3 as the standby router.
-OSPF Area 0 provides dynamic routing between R1, R2, and R3.
-R2 connects the internal routing infrastructure to OPNsense.
-R2 advertises the default route through OSPF.
-EtherChannel provides aggregated links between switches.
-Rapid PVST+ provides Layer 2 loop prevention.

## Redundancy and High Availability

The network is designed to provide fault tolerance and minimize service interruptions through multiple redundancy mechanisms.

- **HSRP:** Provides redundant default gateways for end devices.
- **OSPF:** Enables dynamic routing and network convergence between routers.
- **EtherChannel:** Aggregates multiple physical links into a single logical connection, increasing bandwidth and link redundancy.
- **Rapid PVST+:** Prevents Layer 2 loops while providing fast spanning-tree convergence.

## Security Design

Multiple security mechanisms are implemented to protect the infrastructure against common Layer 2 and Layer 3 threats.

- **DHCP Snooping:** Prevents unauthorized DHCP servers from assigning IP addresses.
- **Dynamic ARP Inspection (DAI):** Helps protect against ARP spoofing attacks.
- **Port Security:** Restricts the number of MAC addresses allowed on access ports.
- **Extended ACLs:** Control traffic between network segments according to security policies.
- **SSHv2:** Provides encrypted remote management.
- **Fail2ban:** Monitors SSH authentication failures and temporarily blocks suspicious IP addresses.
- **Service Hardening:** Disables unnecessary services to reduce the attack surface.

## Fedora Network Services

The Fedora server (`10.10.20.50`) provides centralized network services for the lab.

### DHCP – Kea

DHCP address pools are configured for the following VLANs:

| VLAN | Name |
|---|---|
| 10 | UTENTI |
| 20 | SERVERS |
| 30 | OSPITI |

DHCP Relay is configured on R1 and R3 to forward client requests to the centralized DHCP server.

### DNS – BIND

- **Internal domain:** `azienda.local`
- **DNS server:** `10.10.20.50`
- **Forwarders:** `1.1.1.1`, `8.8.8.8`

BIND provides forward and reverse DNS resolution for internal hosts.

### NFS – Network File Sharing

- **Exported directory:** `/srv/nas`

**Export access policy:**

| Network | Permissions |
|---|---|
| VLAN 20 – `10.10.20.0/24` | Read / Write |
| VLAN 10 – `10.10.10.0/24` | Read-only |
| Other networks | No access configured |

<sub>Client connectivity and permission testing are pending.</sub>

### Centralized Logging – rsyslog

The Fedora server collects Syslog messages from the following network devices:

- Routers: R1, R2, R3
- Switches: SW1, SW2, SW3
- Firewall: OPNsense

Logs are stored in separate files for each device, allowing centralized monitoring and troubleshooting.

### NTP – Network Time Protocol

- **NTP server:** R1
- **Authentication:** Enabled

Network devices synchronize their clocks with R1 using authenticated NTP.

### SSH and Fail2ban

SSH is enabled for remote server administration.

Fail2ban protects the SSH service against repeated failed authentication attempts.

**SSH jail configuration:**

| Parameter | Value |
|---|---:|
| Maximum retries (`maxretry`) | 3 |
| Find time (`findtime`) | 300 seconds |
| Ban time (`bantime`) | 600 seconds |

Fail2ban monitors authentication failures and temporarily bans IP addresses that exceed the configured retry limit.  

## Repository Structure

The repository is organized into separate directories containing device configurations, network documentation, and technology-specific implementations.

Router and switch configurations are maintained separately to simplify configuration management and troubleshooting.

## Project Status

<badge color="success">Implemented</badge> Core routing, switching, and network services configuration.

<badge color="success">Implemented</badge> Fedora DHCP, DNS, Syslog, SSH, and Fail2ban configuration.

<badge color="success">Implemented</badge> OPNsense integration and network firewall configuration.

<badge color="warning">In progress</badge> NFS client connectivity and access permission testing.

<badge color="warning">In progress</badge> NFSv4-only configuration verification.

The network is currently under development. Additional configurations, topology diagrams, technical documentation, and connectivity/security validation tests will be added as the project evolves.
