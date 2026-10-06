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

<img width="1021" height="765" alt="изображение" src="https://github.com/user-attachments/assets/259323ce-8239-4a9a-8e61-3faf442a8ca6" />


<br>

### **2.2 Stage 2: Build**

#### **1. Build a topology in cisco packet tracer**

<br>

<img width="1621" height="1132" alt="изображение" src="https://github.com/user-attachments/assets/26d8112c-ab31-44a7-8eaa-e167e193d7af" />


<br>
<br>
<br>
Configure IP adresses for each PC

<br>
<br>

<img width="1040" height="373" alt="изображение" src="https://github.com/user-attachments/assets/7c96d8b2-c335-4d76-b3ff-e43694c926ce" />




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

<br>

Following that, we can see that all VLANs created successfully using `show vlan brief`

<br>

<img width="850" height="348" alt="изображение" src="https://github.com/user-attachments/assets/976a177d-dce7-4cee-9339-adcd1290d682" />

#### **3 Assign Physical Ports to VLANs**

Now, we have to map VLANs to physical interfaces  on the switch

**Assign Ports fa0/1-2** to **VLAN 10**

<br>

```text
Switch(config)#interface range fastEthernet 0/1-2

Switch(config-if-range)#switchport mode access

Switch(config-if-range)#switchport access vlan 10


Switch(config-if-range)#exit
```

<br>

<img width="531" height="83" alt="изображение" src="https://github.com/user-attachments/assets/8b5b7776-879d-4be0-8cf7-a4734ac575f3" />

<br>

**Assign Ports fa0/3-4** to **VLAN 20**

```text
Switch(config)#interface range fastEthernet 0/3-4

Switch(config-if-range)#switchport mode access

Switch(config-if-range)#switchport access vlan 20

Switch(config-if-range)#exit
```


<br>

<img width="534" height="81" alt="изображение" src="https://github.com/user-attachments/assets/e4941342-7402-4d09-9f02-cf8793302705" />

<br>



**Assign Ports fa0/5-6** to **VLAN 30**

```text
Switch(config)#interface range fastEthernet 0/5-6

Switch(config-if-range)#switchport mode access

Switch(config-if-range)#switchport access vlan 30

Switch(config-if-range)#exit
```

<img width="534" height="79" alt="изображение" src="https://github.com/user-attachments/assets/63bfd15d-4929-48c5-bbb0-723534001b68" />

#### **4. Verify the Configuration:**

Exit configuration mode and check the VLAN database to ensure your ports are assigned correctly with command `show vlan brief`

<br>

<img width="619" height="62" alt="изображение" src="https://github.com/user-attachments/assets/8285eec8-c278-4ca7-83cb-3aa7ec5db40a" />

<br>

#### **Testing with Ping**

PC1 -> PC2 (Same VLAN 10) => ✅Successful

<br>

<img width="638" height="442" alt="изображение" src="https://github.com/user-attachments/assets/5bdbe072-51d6-42c8-86fb-76f93a41b830" />

PC2 -> PC3 (Different: VLAN 10 and VLAN 20) => ❌Failed

<br>

<img width="674" height="425" alt="изображение" src="https://github.com/user-attachments/assets/80195860-dcc8-4bf8-97a9-a897a717969b" />

<br>


**Commands Used:**


| **Command**                           | **Short Version** | **Executed in Mode**                    | **Purpose**                                                                                                    |
| ------------------------------------- | ----------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `enable`                              | `en`              | User EXEC (`Switch>`)                   | Enters Privileged EXEC mode, allowing to view configurations and execute more advanced commands.               |
| `configure terminal`                  | `conf t`          | Privileged EXEC (`Switch#`)             | Enters Global Configuration Mode, where changes affect the entier switch, rather that a specific port          |
| `interface [type][port]`              | `int f0/1`        | Global Config (`Switch(config)#`)       | Enters interface configuration mode for a single specific port                                                 |
| `interface range [type][port]-[port]` | `int f0/1-5`      | Global Config (`Switch(config)#`)       | Enters interface configuration mode for a group of ports<br>                                                   |
| `vlan [id]`                           |                   | Global Config (`Switch(config)#`)       | Creates a new VLAN with the specified ID number and enters VLAN configuration mode                             |
| `name [name]`                         |                   | VLAN Config (`Switch(config-vlan)#`)    | Assigns human-readable name for the VLAN ID                                                                    |
| `switchport mode access`              | `sw mo ac`        | Interface Config (`Switch(config-if)#`) | Forces the port to operate as an access port (connecting to a single end device) and prevents it from trunking |
| `switchport access vlan [id]`         | `sw ac vlan 10`   | Interface Config (`Switch(config-if)#`) | Assigns the access port to the specified VLAN                                                                  |
| `show vlan brief`                     | `sh vlan br`      | Privileged EXEC (`Switch#`)             | Displays the table of all created VLANs, their names, status and which access ports are assigned to them       |
