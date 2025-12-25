# Distribution Switch 1 (DSW1) Configuration

---

**Role:** HSRP Active (Primary Gateway) | STP Root Bridge
## 1. Change Name
```
configure terminal
hostname DSW1
```
---

## 2. Enable Routing Capabilities
```
ip routing
ipv6 unicast-routing
```
---

## 3. VLAN Database
```
vlan 10
 name DEPT-SALES
vlan 20
 name DEPT-HR
vlan 99
 name MGT-NATIVE
exit
```
---

## 4. Spanning Tree (Root Bridge for these VLANs)
```
spanning-tree mode rapid-pvst
spanning-tree vlan 10,20,99 priority 24576
```
---

## 5. Access Ports (PC Connections)
```
interface GigabitEthernet1/0/6
 description Access port to PC1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
 exit
```
---

## 6. Port-Channel (Trunk to DSW2)
```
interface range GigabitEthernet1/0/23-24
 description EtherChannel Trunk to DSW2
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 channel-group 1 mode active
 no shutdown
 exit
```
---

## 7. SVI Configuration (Gateway for VLANs)

VLAN 10 (HSRP Active)
```
interface vlan 10
 description Gateway SVI for Sales
 ip address 192.168.10.2 255.255.255.0
 ipv6 address 2001:db8:acad:10::2/64
 ! DHCP Helper (Forward requests to Windows Server)
 ip helper-address 192.168.50.10 
 ! HSRP Settings
 standby version 2
 standby 10 ip 192.168.10.1
 standby 10 priority 150
 standby 10 preempt
 no shutdown
 exit
```
VLAN 20 (HSRP Active)
```
interface vlan 20
 description Gateway SVI for HR
 ip address 192.168.20.2 255.255.255.0
 ipv6 address 2001:db8:acad:20::2/64
 ip helper-address 192.168.50.10
 ! HSRP Settings
 standby version 2
 standby 20 ip 192.168.20.1
 standby 20 priority 150
 standby 20 preempt
 no shutdown
 exit
```
---

## 8. Uplink to Core Router (Routed Port)
```
interface GigabitEthernet0/0/1
 description Link to R1 (G0/0/1)
 no switchport
 ip address 10.10.10.2 255.255.255.252
 ipv6 address 2001:db8:acad:a::2/64
 no shutdown
 exit
```
---

## 8. Routing Protocol (OSPF)
```
router ospf 10
 router-id 2.2.2.2
 ! Advertise Internal VLANs
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 ! Advertise Uplink
 network 10.10.10.0 0.0.0.3 area 0
 exit
```
```
end
write memory
```

