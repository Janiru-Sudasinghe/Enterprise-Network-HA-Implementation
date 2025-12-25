
---

### 📄 File 2: Name this `L3-Switch-DSW2.md`

```markdown
# Distribution Switch 2 (DSW2) Configuration
**Role:** HSRP Standby (Backup Gateway) | STP Secondary Root

```cisco
! -----------------------------------------------------------------
! DEVICE: Distribution Switch 2 (DSW2)
! -----------------------------------------------------------------

configure terminal
hostname DSW2

! 1. Enable Routing Capabilities
ip routing
ipv6 unicast-routing

! 2. VLAN Database
vlan 10
 name DEPT-SALES
vlan 20
 name DEPT-HR
vlan 99
 name MGT-NATIVE
exit

! 3. Spanning Tree (Backup Root Bridge)
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,99 priority 28672

! 4. Access Ports (PC Connections)
interface GigabitEthernet1/0/6
 description Access port to PC2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
 exit

! 5. Port-Channel (Trunk to DSW1)
interface range GigabitEthernet1/0/23-24
 description EtherChannel Trunk to DSW1
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit

! 6. SVI Configuration (Gateway for VLANs)
! --- VLAN 10 (HSRP Standby) ---
interface vlan 10
 description Gateway SVI for Sales
 ip address 192.168.10.3 255.255.255.0
 ipv6 address 2001:db8:acad:10::3/64
 ip helper-address 192.168.50.10 
 ! HSRP Settings (Lower Priority = Backup)
 standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 100
 no shutdown
 exit

! --- VLAN 20 (HSRP Standby) ---
interface vlan 20
 description Gateway SVI for HR
 ip address 192.168.20.3 255.255.255.0
 ipv6 address 2001:db8:acad:20::3/64
 ip helper-address 192.168.50.10
 ! HSRP Settings
 standby version 2
 standby 20 ip 192.168.20.1
 standby 20 priority 100
 no shutdown
 exit

! 7. Uplink to Core Router (Routed Port)
interface GigabitEthernet0/0/1
 description Link to R1 (G0/0/2)
 no switchport
 ! Different Subnet for Redundancy
 ip address 10.10.11.2 255.255.255.252
 ipv6 address 2001:db8:acad:b::2/64
 no shutdown
 exit

! 8. Routing Protocol (OSPF)
router ospf 10
 router-id 3.3.3.3
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 10.10.11.0 0.0.0.3 area 0
 exit

end
write memory