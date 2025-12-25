# Enterprise Network Infrastructure Implementation

![Status](https://img.shields.io/badge/Status-Completed-success)
![Platform](https://img.shields.io/badge/Platform-Cisco_IOS_%7C_Windows_Server-blue)
![Virtualization](https://img.shields.io/badge/Virtualization-VMware_Workstation-orange)

## 📌 Project Overview
This project involves the design and physical implementation of a scalable enterprise network infrastructure. It simulates a corporate environment requiring High Availability (HA), security, and segmented traffic management.

The implementation includes the configuration of physical Cisco hardware (Routers and Switches) integrated with a virtualized server environment (VMware) to provide core network services like DHCP and DNS.

---

## 🏗️ Architecture & Topology

<p align="center">
  <img src="./Images/network_diagram.jpg" width="600" alt="Network Topology Diagram">
</p>

### Hardware Components
* **Routing:** 1x Cisco ISR Router (Main Gateway)
* **Core/Distribution:** 2x Cisco Layer 3 Switches
* **Access:** 2x Cisco Layer 2 Switches
* **Endpoints:** 3x Host PCs (Simulating end-users)
* **Server:** VMware Workstation Host running Windows Server 2019

---

## ⚙️ Key Configurations & Technologies

### 1. Switching & VLANs
* **VLAN Segmentation:** Created distinct broadcast domains for Departments (HR, IT, Sales) and Management.
* **VTP (VLAN Trunking Protocol):** Configured for centralized VLAN management across the switched fabric.
* **EtherChannel (LACP):** Implemented link aggregation between Layer 3 and Layer 2 switches to increase bandwidth and redundancy.
* **Port Security:** Enabled sticky MAC addressing to prevent unauthorized access at the edge.

### 2. Routing & High Availability
* **Inter-VLAN Routing:** Configured SVI (Switch Virtual Interfaces) on Layer 3 switches.
* **EIGRP:** Dynamic routing protocol implementation for internal network reachability.
* **HSRP (Hot Standby Router Protocol):** Configured on L3 switches to provide default gateway redundancy and failover capabilities.
* **NAT (Network Address Translation):** Configured overload (PAT) for internet access.

### 3. Services & Virtualization
* **Hybrid Cloud Setup:** Integrated physical network with VMware Workstation Pro.
* **DHCP Relay:** Windows Server configured as the centralized DHCP server; Helper addresses configured on SVI interfaces to relay requests across VLANs.
* **Management:** SSH enabled for secure remote management; Syslog and TFTP servers configured for logging and backup.
* **ACLs:** Extended Access Control Lists implemented to restrict traffic flow between specific departments and servers.

### 📸 Physical Implementation Gallery

<p align="center">
  <img src="./images/Rack-Setup-Front.jpg" width="700" alt="Front Rack Setup">
  <br>
  <em>Figure 1: Front view of the rack configuration showing the Cisco ISR Router and Catalyst switches.</em>
</p>

<p align="center">
  <img src="./images/Cabling-Rear.jpg" width="700" alt="Rear Cabling Detail">
  <br>
  <em>Figure 2: Detailed view of the structured cabling and console connections at the rear of the stack.</em>
</p>

## 🚀 How to Use
The configuration files located in the `/configs` folder are sanitized outputs from the running-config of each device.
1.  **Review Logic:** Open any `.txt` file to view the command hierarchy.
2.  **Deployment:** These configs can be loaded onto compatible Cisco hardware or simulated in Packet Tracer/GNS3 (with minor interface adjustments).

## 🏆 Learning Outcomes
* Deployed a robust hierarchical network model (Core, Distribution, Access).
* Mastered physical layer troubleshooting (cabling, interface status, duplex mismatches).
* Successfully integrated virtualized server services (DHCP/DNS) into a physical routing domain.

---
*Created by [Your Name]*
