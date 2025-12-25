
# Access Switch 2 (ASW2) Configuration

---
**Role:** Access Layer for HR Dept | Uplink to Distribution

## 1. Basic Device Hardening & Settings
```
configure terminal
hostname ASW2
enable secret class
no ip domain-lookup
service password-encryption

! Banner
banner motd #Authorized Access Only!#
```
---

## 2. SSH Management Configuration
```
ip domain-name western-hotel.local
crypto key generate rsa modulus 2048
username admin secret ccna
ip ssh version 2
line vty 0 15
 transport input ssh
 login local
 exit
```
---

## 3. VLAN Database Creation
```
vlan 10
 name DEPT-SALES
vlan 20
 name DEPT-HR
vlan 150
 name VOICE-VLAN
vlan 99
 name MGT-NATIVE
exit
```
---

## 4. Management Interface (SVI)
```
interface vlan 99
 description Management Interface
 ! Unique IP for ASW2
 ip address 192.168.99.12 255.255.255.0
 no shutdown
 exit
! Default Gateway
ip default-gateway 192.168.99.1
```
---

## 5. EtherChannel Uplinks to Distribution Layer
```
interface range GigabitEthernet0/1 - 2
 description Uplink Trunk to DSW Layer
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,150,99
 channel-group 1 mode active
 ! Security Trusts for Uplinks
 ip dhcp snooping trust
 ip arp inspection trust
 no shutdown
 exit
```
---

## 6. Access Ports (HR Department + Phones)
```
interface range FastEthernet0/1 - 24
 description Access Ports for HR PC & Phones
 switchport mode access
 ! HR uses VLAN 20
 switchport access vlan 20
 switchport voice vlan 150
 ! Performance Tuning
 spanning-tree portfast
 spanning-tree bpduguard enable
 ! Port Security
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
 switchport port-security aging time 10
 switchport port-security aging type inactivity
 no shutdown
 exit
```
---

## 7. L2 Security Features (Global)
```
ip dhcp snooping
ip dhcp snooping vlan 10,20,150,99
! DAI (Dynamic ARP Inspection)
ip arp inspection vlan 10,20,150,99
ip arp inspection validate src-mac dst-mac ip
```
```
end
write memory
```
