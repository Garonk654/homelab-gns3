# Enterprise Homelab Network

A multi-layer enterprise network lab designed to simulate a small corporate infrastructure. This project focuses on implementing enterprise networking technologies, network segmentation, redundancy, dynamic routing, and security mechanisms using Cisco routers and switches.

## Project Overview

This project consists of designing, implementing, and documenting a small enterprise network in a virtualized environment. It brings together routing, switching, network security, firewalling, and centralized Linux services to replicate the core components of a corporate IT infrastructure.

### Lab Environment & Tools

The lab is built using the following platforms:

- GNS3: Network topology design, emulation, and testing of Cisco routers and switches.
- VMware Workstation: Virtualization platform used to run the lab environment and virtual machines.
- Fedora Linux: Server operating system hosting centralized network services.
- OPNsense: perimeter firewall (rules and policies configured; WAN/NAT not testable in the isolated lab)

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
- Syslog
- FAIL2BAN

  
### Routing and Switching Architecture

- R1 and R3 provide inter-VLAN routing through Router-on-a-Stick.
- HSRP provides default gateway redundancy, with R1 configured as the primary router and R3 as the standby router.
- OSPF Area 0 provides dynamic routing between R1, R2, and R3.
- R2 connects the internal routing infrastructure to OPNsense.
- R2 advertises the default route through OSPF.
- EtherChannel provides aggregated links between switches.
- Rapid PVST+ provides Layer 2 loop prevention.

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

## Security Policy

Traffic control is split in two layers:

- **East-west (between VLANs)** – extended ACLs on R1 and R3:
  - Guests (VLAN 30): DHCP, DNS to the Fedora server only, internet access.
    No access to internal networks or router loopbacks.
  - Users (VLAN 10): DHCP, DNS, NTP, NFS and SMB towards Fedora only,
    no management access, internet access.
- **North-south (towards outside)** – OPNsense perimeter firewall rules.

Fedora is managed only through SSH; The ACLs do not expose SSH or other management ports to users or guests.

### Dynamic ARP Inspection and statically addressed hosts

Dynamic ARP Inspection (DAI) validates ARP packets against the DHCP snooping
binding table. In this lab the client PCs (VPCS) and the Fedora server use
static IP addresses, so they never obtain a DHCP lease and do not appear in
that table. Without extra configuration, DAI would drop their ARP packets and
they would lose connectivity.

Two solutions were used:

- **VPCS clients:** static entries were added to the DHCP snooping binding
  table on the access switch of each PC.
  Example:
  `ip dhcp snooping binding 0050.7966.6802 vlan 10 10.10.10.3 interface gi1/2 expiry 4294967295`
- **Fedora server:** an ARP ACL on SW1(`SERVER_STATICI`) maps the server IP
  (10.10.20.50) to its MAC address and is applied to VLAN 20 with
  `ip arp inspection filter`.

Inter-switch trunks and router-facing ports are configured as trusted for DAI,
while access ports stay untrusted. Static bindings are not stored in the
running configuration, so they are listed in `configs/` and must be re-applied
if the switches are reloaded without a snooping database.

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

The zone contains the following records:

| Name | Address | Notes |
|---|---|---|
| `ns1.azienda.local` | 10.10.20.50 | Fedora server (real host) |
| `dc.azienda.local` | 10.10.20.60 | Example record |
| `server.azienda.local` | 10.10.20.70 | Example record |
| `nas.azienda.local` | 10.10.20.80 | Example record |

The records `dc`, `server`, `nas` are examples used to test
name resolution: no hosts with those names exist in the lab.

The DNS domain `azienda.local` is used for the Linux services only. The Cisco
devices use a different domain (`homelab`), configured with `ip domain-name`
for SSH key generation, and it is not related to the DNS zone.

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

- ✅ Core routing, switching and network services
- ✅ Fedora services: DHCP, DNS, Syslog, SSH, Fail2ban
- ✅ OPNsense rules and policies configured
- 🔄 NFS client mount and permission tests (pending)
- 🔄 NFSv4-only verification (pending)

The network is currently under development. Additional configurations, topology diagrams, technical documentation, and connectivity/security validation tests will be added as the project evolves.

## Known Limitations

- The OPNsense WAN has no address, so NAT and internet access
  through the firewall are configured but not testable.
- The firewall is perimetral only: inter-VLAN traffic is routed by R1/R3
  and filtered by ACLs, it does not cross the firewall.
- Guest-to-guest traffic inside VLAN 30 is not filtered.
- Lab credentials and keys (enable secret, admin user, OSPF/NTP keys)
  are lab-only values, not meant for production.
