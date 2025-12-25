# Cisco IOS Configuration Reference Guide

## 1. Device Management & Hardening
Basic router initialisation, password encryption, and administrative access.

```cisco
! Configure Hostname and Enable Secret
configure terminal
hostname R1
enable secret class

! Console Security
line console 0
 password cisco
 login
 exit

! VTY (Telnet/SSH) Security
line vty 0 4
 password cisco
 login
 exit

! Service Encryption & Banner
service password-encryption
banner motd # Authorised Access Only!#

! Command History Buffer
terminal history size 200
show history

---

## 2. Interface Configuration (IPv4 & IPv6)
Configuration for physical interfaces, loopbacks, and VLAN tagging (Router-on-a-Stick).

! Physical Interface Configuration
interface gigabitethernet 0/0/0
 description Link to LAN 1
 ip address 192.168.10.1 255.255.255.0
 ipv6 address 2001:db8:acad:1::1/64
 no shutdown
 exit

! Loopback Interface
interface loopback 0
 ip address 1.1.1.1 255.255.255.255
 exit

! Subinterface (Router-on-a-Stick) for VLAN 10
interface G0/0/1.10
 description Default Gateway for VLAN 10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

! Enable Physical Parent Interface
interface G0/0/1
 description Trunk link to S1
 no shutdown
 end

---

## 3. IPv6 Services (DHCPv6)
Implementations for Stateless (SLAAC) and Stateful DHCPv6 servers and clients.

! Enable IPv6 Routing globally
ipv6 unicast-routing

! --- Option A: Stateless DHCPv6 Server (SLAAC + Other Info) ---
ipv6 dhcp pool IPV6-STATELESS
 dns-server 2001:db8:acad:1::254
 domain-name example.com
 exit

interface GigabitEthernet0/0/1
 description Link to LAN
 ipv6 address fe80::1 link-local
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 nd other-config-flag
 ipv6 dhcp server IPV6-STATELESS
 no shutdown
 exit

! --- Option B: Stateful DHCPv6 Server (Address + Other Info) ---
ipv6 dhcp pool IPV6-STATEFUL
 address prefix 2001:db8:acad:1::/64
 dns-server 2001:4860:4860::8888
 domain-name example.com
 exit

interface GigabitEthernet0/0/1
 ipv6 address fe80::1 link-local
 ipv6 address 2001:db8:acad:1::1/64
 ipv6 nd managed-config-flag
 ipv6 nd prefix default no-autoconfig
 ipv6 dhcp server IPV6-STATEFUL
 exit

! DHCPv6 Relay Agent
interface gigabitethernet 0/0/1
 ipv6 dhcp relay destination 2001:db8:acad:1::2 G0/0/0
 exit

! Client Configuration (On a different router, e.g., R3)
interface g0/0/1
 ipv6 enable
 ipv6 address autoconfig  ! For Stateless
 ! OR
 ipv6 address dhcp        ! For Stateful
 end

---

## 4. Routing Protocols
Static routing, Floating static routes, and OSPF configuration.

! --- IPv4 Static Routing ---
! Next-Hop IP
ip route 172.16.1.0 255.255.255.0 172.16.2.2
! Exit Interface
ip route 172.16.1.0 255.255.255.0 s0/1/0
! Fully Specified
ip route 172.16.1.0 255.255.255.0 GigabitEthernet 0/0/1 172.16.2.2
! Default Route
ip route 0.0.0.0 0.0.0.0 172.16.2.2
! Floating Static Route (Backup link with AD 5)
ip route 0.0.0.0 0.0.0.0 10.10.10.2 5

! --- IPv6 Static Routing ---
ipv6 route 2001:db8:acad:1::/64 2001:db8:acad:2::2
ipv6 route ::/0 2001:db8:acad:2::2
! Floating IPv6 Route
ipv6 route ::/0 2001:db8:feed:10::2 5

! --- Host Routes ---
ip route 209.165.200.238 255.255.255.255 198.51.100.2
ipv6 route 2001:db8:acad:2::238/128 2001:db8:acad:1::2

! --- OSPF Configuration ---
router ospf 10
 network 10.10.1.0 0.0.0.255 area 0
 network 10.1.1.4 0.0.0.3 area 0
 network 10.1.1.12 0.0.0.3 area 0
 exit

! OSPF Priority (Dr/BDr Election)
interface GigabitEthernet 0/0/0
 ip ospf priority 255
 exit

--- 

## 5. First Hop Redundancy (HSRP)
High Availability configuration for gateway redundancy.

interface g0/1
 standby version 2
 standby 1 ip 192.168.1.254
 standby 1 priority 150
 standby 1 preempt
 exit

---

## 6. Security (ACLs)
Standard and Extended Access Control Lists for traffic filtering and line security.

! Numbered Standard ACL (Permit specific host)
access-list 10 remark ACE permits ONLY host 192.168.10.10
access-list 10 permit host 192.168.10.10
access-list 10 permit 192.168.20.0 0.0.0.255
! Applying to Interface
interface Serial 0/1/0
 ip access-group 10 out
 exit

! Named Standard ACL
ip access-list standard PERMIT-ACCESS
 remark ACE permits host 192.168.10.10
 permit host 192.168.10.10
 permit 192.168.20.0 0.0.0.255
 exit

! Secure VTY (SSH/Telnet) with ACL
line vty 0 4
 access-class ADMIN-HOST in
 exit

! Extended ACL (Web Traffic)
access-list 110 permit tcp 192.168.10.0 0.0.0.255 any eq www
access-list 110 permit tcp 192.168.10.0 0.0.0.255 any eq 443
interface g0/0/0
 ip access-group 110 in
 exit

! TCP Established (Allow return traffic)
access-list 120 permit tcp any 192.168.10.0 0.0.0.255 established
interface g0/0/0
 ip access-group 120 out
 exit

---

## 7. Network Address Translation (NAT)
Static, Dynamic, and PAT (Port Address Translation) configurations.

! --- Static NAT ---
! Map internal server to public IP
ip nat inside source static 192.168.10.254 209.165.201.5

! --- Dynamic NAT ---
! Define Pool
ip nat pool NAT-POOL1 209.165.200.226 209.165.200.240 netmask 255.255.255.224
! Define Traffic
access-list 1 permit 192.168.0.0 0.0.255.255
! Map Traffic to Pool
ip nat inside source list 1 pool NAT-POOL1

! --- PAT (Overload) using Interface IP ---
ip nat inside source list 1 interface serial 0/1/1 overload

! --- PAT (Overload) using Pool ---
ip nat pool NAT-POOL2 209.165.200.226 209.165.200.240 netmask 255.255.255.224
ip nat inside source list 1 pool NAT-POOL2 overload

! --- Interface Application (Required for all NAT types) ---
interface serial 0/1/0
 ip nat inside
 exit
interface serial 0/1/1
 ip nat outside
 exit

---




