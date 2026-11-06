# SW1 – Switch Configuration

## 1. Device Overview

| Parameter | Value |
|---|---|
| Hostname | SW1 |
| IOS Version | 15.2 |
| Management VLAN | VLAN 99 |
| Management IP | 10.10.99.10/24 |
| Default Gateway | 10.10.99.254 |
| Spanning Tree | Rapid PVST+ |
| STP Role | Root Bridge |
| EtherChannel | LACP |
| DHCP Snooping | Enabled |
| Dynamic ARP Inspection | Enabled |

SW1 is the primary Layer 2 switch in the topology. It acts as the STP root bridge for the configured VLANs and provides Layer 2 connectivity through trunk links and EtherChannel.

## 2. Complete Configuration

The following configuration is based on the actual `show running-config` output. Passwords and cryptographic keys have been redacted.

```cisco
!
! ==============================
! SYSTEM CONFIGURATION
! ==============================
!
version 15.2
service timestamps debug datetime msec
service timestamps log datetime msec
service password-encryption
service compress-config
!
hostname SW1
!
logging buffered 16384 informational
enable secret 5 <REDACTED>
username admin privilege 15 secret 5 <REDACTED>
!
no aaa new-model
!
clock timezone CET 1 0
clock summer-time CEST recurring last Sun Mar 2:00 last Sun Oct 3:00
!
no ip routing
no ip domain-lookup
ip domain-name homelab
no ip cef
no ipv6 cef
!
no cdp run
!
! ==============================
! DHCP SNOOPING
! ==============================
!
ip dhcp snooping vlan 10,20,30,99
no ip dhcp snooping information option
ip dhcp snooping
!
! ==============================
! DYNAMIC ARP INSPECTION
! ==============================
!
ip arp inspection vlan 10,20,30,99
ip arp inspection validate src-mac dst-mac ip
ip arp inspection filter SERVER_STATICI vlan 20
!
arp access-list SERVER_STATICI
 permit ip host 10.10.20.50 mac host 000c.29c8.b613
!
! ==============================
! ERROR-DISABLE RECOVERY
! ==============================
!
errdisable recovery cause bpduguard
errdisable recovery interval 60
!
! ==============================
! SPANNING TREE
! ==============================
!
spanning-tree mode rapid-pvst
spanning-tree portfast edge default
spanning-tree portfast edge bpduguard default
spanning-tree extend system-id
spanning-tree pathcost method long
spanning-tree vlan 10,20,30,99 priority 24576
!
! ==============================
! ETHERCHANNEL - PORT-CHANNEL 1
! ==============================
!
interface Port-channel1
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 spanning-tree guard root
!
! ==============================
! ETHERCHANNEL - PORT-CHANNEL 2
! ==============================
!
interface Port-channel2
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 spanning-tree guard root
!
! ==============================
! TRUNK AND ETHERCHANNEL PORTS
! ==============================
!
interface GigabitEthernet0/0
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 negotiation auto
 ip dhcp snooping trust
!
interface GigabitEthernet0/1
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 negotiation auto
 channel-group 1 mode active
!
interface GigabitEthernet0/2
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 negotiation auto
 channel-group 2 mode active
!
interface GigabitEthernet0/3
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 negotiation auto
 channel-group 1 mode active
!
interface GigabitEthernet1/0
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 negotiation auto
 channel-group 2 mode active
!
interface GigabitEthernet1/1
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 999
 switchport mode trunk
 switchport nonegotiate
 ip arp inspection trust
 negotiation auto
 ip dhcp snooping trust
!
! ==============================
! ACCESS PORTS
! ==============================
!
interface GigabitEthernet1/3
 switchport access vlan 20
 switchport mode access
 switchport nonegotiate
 negotiation auto
 ip dhcp snooping trust
!
interface GigabitEthernet2/0
 switchport access vlan 20
 switchport mode access
 negotiation auto
!
! ==============================
! UNUSED PORTS
! ==============================
!
interface GigabitEthernet1/2
 description PORTA INUTILIZZATA
 switchport trunk native vlan 999
 shutdown
 negotiation auto
!
interface GigabitEthernet2/1
 description PORTA INUTILIZZATA
 shutdown
 negotiation auto
!
interface GigabitEthernet2/2
 description PORTA INUTILIZZATA
 shutdown
 negotiation auto
!
interface GigabitEthernet2/3
 description PORTA INUTILIZZATA
 shutdown
 negotiation auto
!
interface GigabitEthernet3/0
 description PORTA INUTILIZZATA
 shutdown
 negotiation auto
!
interface GigabitEthernet3/1
 description PORTA INUTILIZZATA
 shutdown
 negotiation auto
!
interface GigabitEthernet3/2
 description PORTA INUTILIZZATA
 shutdown
 negotiation auto
!
interface GigabitEthernet3/3
 description PORTA INUTILIZZATA
 shutdown
 negotiation auto
!
! ==============================
! MANAGEMENT INTERFACE
! ==============================
!
interface Vlan99
 ip address 10.10.99.10 255.255.255.0
 no ip route-cache
!
ip default-gateway 10.10.99.254
!
! ==============================
! SSH AND HTTP SERVICES
! ==============================
!
no ip http server
ip http secure-server
ip ssh version 2
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr
!
! ==============================
! SYSLOG
! ==============================
!
logging origin-id hostname
logging source-interface Vlan99
logging host 10.10.20.50
!
! ==============================
! NTP
! ==============================
!
ntp authentication-key 1 md5 <REDACTED>
ntp authenticate
ntp trusted-key 1
ntp server 1.1.1.1 key 1
!
! ==============================
! MANAGEMENT ACCESS
! ==============================
!
line con 0
 logging synchronous
!
line aux 0
!
line vty 0 4
 exec-timeout 0 0
 login local
 transport input ssh
!
! ==============================
! BANNER
! ==============================
!
banner motd ^C SOLO ACCESSO AUTORIZZATO ^C
!
end
```

## 3. Configuration Summary

| Feature | Implementation |
|---|---|
| Switching | Layer 2 |
| VLANs | 10, 20, 30, 99 |
| Native VLAN | 999 |
| Spanning Tree | Rapid PVST+ |
| STP Root Bridge | SW1 |
| EtherChannel | LACP |
| DHCP Protection | DHCP Snooping |
| ARP Protection | Dynamic ARP Inspection |
| Static ARP Binding | SERVER_STATICI |
| Management | VLAN 99 |
| Remote Access | SSH v2 |
| Centralized Logging | Syslog |
| Time Synchronization | NTP |

## 4. Spanning Tree Configuration

SW1 is configured as the primary root bridge for VLANs 10, 20, 30 and 99.

The configured priority is 24576, lower than the default priority of 32768.

Additional protections:
- Rapid PVST+ is enabled.
- PortFast Edge is enabled globally by default.
- BPDU Guard is enabled globally on PortFast Edge ports.
- Root Guard is configured on Port-channel1 and Port-channel2.
- Errdisable recovery for BPDU Guard is configured with a 60-second interval.

## 5. EtherChannel Configuration

Two EtherChannel groups are configured using LACP active mode.

| Port-Channel | Physical Interfaces |
|---|---|
| Port-channel1 | Gi0/1, Gi0/3 |
| Port-channel2 | Gi0/2, Gi1/0 |

Both Port-channels operate as 802.1Q trunks, carrying VLANs 10, 20, 30 and 99, with VLAN 999 as the native VLAN.

## 6. Layer 2 Security

**DHCP Snooping**

- Enabled globally.
- Active on VLANs 10, 20, 30 and 99.
- DHCP Option 82 insertion disabled.
- Trusted interfaces configured on selected uplinks and server-facing ports.

**Dynamic ARP Inspection**

- Enabled on VLANs 10, 20, 30 and 99.
- Source MAC, destination MAC and IP validation enabled.
- Static ARP ACL `SERVER_STATICI` applied to VLAN 20.
- Static binding configured for Fedora server `10.10.20.50`, MAC `000c.29c8.b613`.

## 7. Management and Monitoring

- Management IP: `10.10.99.10/24`
- Default gateway: `10.10.99.254`
- SSH version 2 enabled.
- HTTPS server enabled; HTTP server disabled.
- Syslog sent to Fedora at `10.10.20.50`.
- Syslog source interface: VLAN 99.
- NTP synchronization with R1 at `1.1.1.1`, using authentication key 1.

## 8. Notes

- SW1 is configured as a Layer 2 switch (`no ip routing`).
- VLAN 999 is used as the native VLAN on configured trunk links.
- VLANs 10, 20, 30 and 99 are explicitly allowed on trunks.
- Two access ports are assigned to VLAN 20.
- Unused interfaces are administratively disabled.
- Credentials and cryptographic keys have been redacted from this public documentation.

**Important:** This document reflects the configuration supplied through `show running-config`. 