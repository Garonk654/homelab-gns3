# Fedora Server

## 1. Device Overview

| Parameter | Value |
|---|---|
| Operating System | Fedora Workstation 44 |
| Role | Infrastructure Services Server |
| Hostname | fedora |
| LAB Interface | ens224 |
| LAB IP Address | 10.10.20.50/24 |
| Lab gateway | 10.10.20.254 (HSRP address of R1/R3) |
| LAB VLAN | VLAN 20 - SERVERS |
| NAT Interface | ens160 |
| NAT IP Address | 192.168.8.131 |

## 2. Network Configuration

The server uses two network interfaces.

| Interface | IP Address | Purpose |
|---|---|---|
| ens160 | 192.168.8.131 | 	Internet access through the VMware NAT |
| ens224 | 10.10.20.50/24 | Lab network, connected to VLAN 20 (SERVERS) |

### Routing

- Internet traffic leaves through `ens160` and the NAT gateway `192.168.8.2`.
- Traffic to the lab networks (`10.10.0.0/16`) leaves through `ens224`, using a static route with next hop `10.10.20.254`.

The static route is needed because, without it, the routing table sends lab traffic out through the NAT interface.

```
sudo nmcli connection modify GNS3-VLAN20 +ipv4.routes "10.10.0.0/16 10.10.20.254"
```

The `GNS3-VLAN20` connection was created with NetworkManager for `ens224`.

## 3. Network Services

The Fedora server provides several infrastructure services for the laboratory.

### 3.1 DHCP Server

- DHCP implementation: Kea DHCP, listening on `ens224`.
- Lease time: 86400 seconds (24 hours).
- Clients in VLANs 10 and 30 reach the server through the DHCP relay (`ip helper-address`) configured on R1 and R3.

| VLAN | Pool | Gateway | DNS server |
|:---:|---|---|---|
| 10 | 10.10.10.100 - 10.10.10.200 | 10.10.10.254 | 10.10.20.50 |
| 20 | 10.10.20.100 - 10.10.20.200 | 10.10.20.254 | 10.10.20.50 |
| 30 | 10.10.30.100 - 10.10.30.200 | 10.10.30.254 | 10.10.20.50 |

The pools start at `.100`, so the static addresses (VPCS test hosts and the server itself) never conflict with leased ones.

### 3.2 DNS Server (BIND)

| Parameter | Value |
|---|---|
| Listens on | 10.10.20.50, port 53 |
| Internal domain | `azienda.local` |
| Forwarders | 1.1.1.1, 8.8.8.8 |
| Recursion | enabled |
| Queries accepted from | VLAN 10, 20, 30 and 99 only |

Forward zone `azienda.local` and reverse zone `20.10.10.in-addr.arpa` are configured.

| Name | Address | Notes |
|---|---|---|
| `ns1.azienda.local` | 10.10.20.50 | The Fedora server itself |
| `dc.azienda.local` | 10.10.20.60 | Example record |
| `server.azienda.local` | 10.10.20.70 | Example record |
| `nas.azienda.local` | 10.10.20.80 | Example record |

The records `dc`, `server` and `nas` are examples used to test name resolution: no hosts with those names exist in the lab.

The Cisco devices use a different domain (`homelab`), set only to generate SSH keys. It is not related to the DNS zone.

### 3.3 Syslog Server (rsyslog)

rsyslog receives log messages on UDP port 514 and writes one file per device in `/var/log/remote/`, selected by the sender IP address.

| Device | Source address | Log file |
|---|---|---|
| R1 | 1.1.1.1 | `R1.log` |
| R2 | 2.2.2.2 | `R2.log` |
| R3 | 3.3.3.3 | `R3.log` |
| SW1 | 10.10.99.10 | `SW1.log` |
| SW2 | 10.10.99.20 | `SW2.log` |
| SW3 | 10.10.99.30 | `SW3.log` |
| OPNsense | 10.10.250.1 | `OPNsense.log` |

Routers use their loopback as the source address and switches use their VLAN 99 address. The firewall sends its logs through R2, so they arrive with the firewall LAN address.

### 3.4 NFS Server

The server shares the directory `/srv/nas`.

| Network | Access |
|---|---|
| VLAN 20 - `10.10.20.0/24` | Read / Write |
| VLAN 10 - `10.10.10.0/24` | Read-only |
| Other networks | Not authorized |

Root on the clients is mapped to an anonymous user (`root_squash`). Guests cannot reach NFS: they are not in the export and the router ACL does not allow it.

Status: NFS client mounts and the NFSv4-only configuration are still to be verified. 

### 3.5 SSH Server

SSH is enabled for remote administration and tested from a lab host.

- The ACLs on R1 and R3 do not allow users (VLAN 10) or guests (VLAN 30) to reach SSH on this server.
- Hosts in VLAN 20 are in the same network segment, so traffic from them does not cross the routers and is not filtered by ACLs.
- Brute-force attempts are limited by Fail2ban (see section 5).

### 3.6 Time synchronization

Fedora does not provide NTP. The NTP server of the lab is router R1.

## 4. Firewall (firewalld)

firewalld is active on the server and allows these services:

| Service | Port |
|---|---|
| SSH | 22/tcp |
| DNS | 53/udp, 53/tcp |
| DHCP | 67/udp |
| Syslog | 514/udp |
| NFS | `nfs` service |

These rules are **not restricted by source address**: access control towards the server is done by the ACLs on R1 and R3 and by the service configuration (for example the NFS export and the DNS `allow-query` list).

## 5. Security Configuration

**Implemented**

- Fail2ban protects SSH: a client is banned for 10 minutes after 3 failed attempts within 5 minutes.
- Router ACLs on R1 and R3 limit what users and guests can reach on this server.
- NFS export restricted by network, with different permissions per VLAN.
- DNS queries accepted only from the lab VLANs.

| Fail2ban parameter | Value |
|---|---|
| Protected service | SSH |
| Max retry (`maxretry`) | 3 |
| Find time (`findtime`) | 300 seconds |
| Ban time (`bantime`) | 600 seconds |

## 6. Role in the Lab

| Service | Used by |
|---|---|
| DHCP | Clients in VLAN 10, 20 and 30 |
| DNS | All VLANs, as the only allowed DNS server for users and guests |
| Syslog | R1, R2, R3, SW1, SW2, SW3 and OPNsense |
| NFS | VLAN 20 (read/write) and VLAN 10 (read-only) |
| SSH | Administration of the server |
