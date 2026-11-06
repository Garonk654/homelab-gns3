# SW3 – Switch Configuration

## 1. Device Overview

| Parameter              | Value          |
| ---------------------- | -------------- |
| Hostname               | SW3            |
| IOS Version            | 15.2           |
| Management VLAN        | VLAN 99        |
| Management IP          | 10.10.99.30/24 |
| Default Gateway        | 10.10.99.254   |
| Spanning Tree          | Rapid PVST+    |
| EtherChannel           | LACP           |
| DHCP Snooping          | Enabled        |
| Dynamic ARP Inspection | Enabled        |
| Port Security          | Enabled        |

SW3 is a Layer 2 switch in the enterprise network topology. It provides access connectivity through VLAN 10 and VLAN 30, and trunk connectivity through two LACP EtherChannels.

## 2. Complete Configuration

The following configuration is based on the actual `show running-config` output.

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
hostname SW3
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
 ip dhcp snooping trust
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
 ip dhcp snooping trust
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
 channel-group 1 mode active
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
 channel-group 2 mode active
 ip dhcp snooping trust
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
 ip dhcp snooping trust
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
 ip dhcp snooping trust
!
! ==============================
! ACCESS PORTS - VLAN 30
! ==============================
!
interface GigabitEthernet1/0
 switchport access vlan 30
 switchport mode access
 switchport nonegotiate
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0050.7966.6804
 switchport port-security
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/1
 switchport access vlan 30
 switchport mode access
 switchport nonegotiate
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
!
interface GigabitEthernet1/2
 switchport access vlan 30
 switchport mode access
 switchport nonegotiate
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
!
! ==============================
! ACCESS PORTS - VLAN 10
! ==============================
!
interface GigabitEthernet1/3
 switchport access vlan 10
 switchport mode access
 switchport nonegotiate
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security mac-address sticky 0050.7966.6807
 switchport port-security
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
!
! ==============================
! UNUSED PORTS
! ==============================
!
interface GigabitEthernet2/0
 description PORTA INUTILIZZATA
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
 ip address 10.10.99.30 255.255.255.0
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
end
```

## 3. Configuration Summary

| Feature              | Implementation         |
| -------------------- | ---------------------- |
| Switching            | Layer 2                |
| VLANs                | 10, 20, 30, 99         |
| Native VLAN          | 999                    |
| Spanning Tree        | Rapid PVST+            |
| EtherChannel         | LACP                   |
| DHCP Protection      | DHCP Snooping          |
| ARP Protection       | Dynamic ARP Inspection |
| Port Security        | Sticky MAC / Restrict  |
| Management           | VLAN 99                |
| Remote Access        | SSH v2                 |
| Centralized Logging  | Syslog                 |
| Time Synchronization | NTP                    |

## 4. Spanning Tree Configuration

SW3 uses Rapid PVST+ for Layer 2 loop prevention.

The running configuration does not explicitly define an STP priority for SW3. Therefore, the default priority applies to the configured VLANs unless modified elsewhere.

Additional protections:

- Rapid PVST+ is enabled.
- PortFast Edge is enabled globally by default.
- BPDU Guard is enabled globally on PortFast Edge ports.
- Errdisable recovery for BPDU Guard is configured with a 60-second interval.

## 5. EtherChannel Configuration

Two EtherChannel groups are configured using LACP active mode.

| Port-Channel  | Physical Interfaces |
| ------------- | ------------------- |
| Port-channel1 | Gi0/0, Gi0/3        |
| Port-channel2 | Gi0/1, Gi0/2        |

Both Port-channels operate as 802.1Q trunks, carrying VLANs 10, 20, 30 and 99, with VLAN 999 as the native VLAN.

DHCP Snooping trust and Dynamic ARP Inspection trust are configured on both Port-channels and their physical member interfaces.

## 6. Layer 2 Security

**DHCP Snooping**

- Enabled globally.
- Active on VLANs 10, 20, 30 and 99.
- DHCP Option 82 insertion disabled.
- EtherChannel member interfaces configured as trusted.

**Dynamic ARP Inspection**

- Enabled on VLANs 10, 20, 30 and 99.
- Source MAC, destination MAC and IP validation enabled.
- Port-channels and their physical member interfaces configured as trusted.

**Port Security**

Port Security is enabled on all four access ports.

- Sticky MAC learning is configured.
- Explicit sticky MAC addresses are present on Gi1/0 and Gi1/3.
- Gi1/1 and Gi1/2 use sticky learning without an explicitly displayed MAC address.
- Violation mode is set to `restrict`.
- PortFast Edge and BPDU Guard are enabled on access ports.

## 7. Management and Monitoring

- Management IP: `10.10.99.30/24`
- Default gateway: `10.10.99.254`
- SSH version 2 enabled.
- HTTPS server enabled; HTTP server disabled.
- Syslog sent to Fedora at `10.10.20.50`.
- Syslog source interface: VLAN 99.
- NTP synchronization with R1 at `1.1.1.1`, using authentication key 1.
