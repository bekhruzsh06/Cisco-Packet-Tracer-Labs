## **1. Theory**

**VLAN:** Logical grouping of network devices into a single broadcast domain on L2, that divides devices into segments and isolates them between each other.

- Allows partitioning a single physical switch into multiple isolated logical switches

**Why use VLANs?**

- **Security:** Devices in VLAN 10 cannot communicate with devices in VLAN 20 without a router or Layer 3 switch. This isolates sensitive departments (like IT) from general stuff

- **Performance:** By dividing a large network into smaller VLANs, you reduce the size of the broadcast domain. A broadcast message sent by one PC won't interrupt every computer in the building

- **Flexibility:** You can group users by functions rather than location. A user from the 1st and user from the 5th floor in the same department can be placed in the same local network



## **2. Cisco Packet Tracer VLAN Setup**

This guide walks through configuring 3 VLANs (VLAN10: Sales, VLAN20: HR, VLAN30: IT) on a single Cisco 2960 switch with 6 PCs


### **2.1 Stage 1: Design**

<br>
<br>

### **2.2 Stage 2: Build**

#### **1. Build a topology in cisco packet tracer**

- Configure IP adresses for each PC

<br>

<img width="1040" height="373" alt="изображение" src="https://github.com/user-attachments/assets/7c96d8b2-c335-4d76-b3ff-e43694c926ce" />

<br>

<img width="1621" height="1132" alt="изображение" src="https://github.com/user-attachments/assets/26d8112c-ab31-44a7-8eaa-e167e193d7af" />


### **2. Create the VLAN on the Switch**

Click the switch to go to CLI tab. Enter global configuration mode to create and name VLANs

<br>

<img width="629" height="216" alt="изображение" src="https://github.com/user-attachments/assets/3eed6335-8341-48bb-9320-2f1647260b2f" />

<br>

<br>

<br>

```Cisco
Switch>enable

Switch#conf t

Enter configuration commands, one per line. End with CNTL/Z.

Switch(config)#vlan 10

Switch(config-vlan)#name HR

Switch(config)#vlan 20

Switch(config-vlan)#name SALES

Switch(config-vlan)#vlan 30

Switch(config-vlan)#name IT
```
