
# Trunk VLAN Configuration

While standard VLAN is great for segmenting a single switch into a multiple isolated broadcast domains, it quickly encounteres scalability issues as network grows larger.

### **Problems with Standard VLANs**


- **Scalability**: Standard access ports carry only one VLAN. Connecting switches that host 50 VLANs would require 50 dedicated cables and 100 switch ports just for hardware linking, consuming ports meant for end-users.
    
- **Cost**: Relying on one-to-one physical links drastically increases the budget required for copper/fiber cabling, switch port real estate, and installation labor.
    
- **Inflexibility**: Network expansion becomes a hardware obstacle. Deploying a new VLAN requires running new physical cables between every switch rather than simply applying a software configuration.


### **The Solution: VLAN Trunking**


**Trunk Port:** A special type of point-to-point connection between network devices, that is designed to carry traffic for **multiple** VLANs with a single physical cable, acting as a multi-lane highway


#### How Trunking Works (802.1Q Encapsulation)

To prevent traffic from different VLANs from mixing together as it travels across the single trunk cable, the sending switch modifies the data.


1. **Tagging:** Before sending an Ethernet frame across the trunk link, the switch inserts a 4-byte 802.1Q "Tag" into the frame's header. This tag contains **VLAN ID** (e.g VLAN 20, or VLAN 10)

2. **Transit:** The tagged frame travels across the single physical cable.
    
3.  **Untagging/Routing:** The receiving device reads the tag, removes it, and forwards the original, untagged frame to the correct destination port within that specific VLAN

<br>

<img width="1071" height="652" alt="изображение" src="https://github.com/user-attachments/assets/a8b776ba-310a-4451-a138-e9669326365b" />

<br>

## **2. Cisco Packet Tracer VLAN Setup**

This guide walks through configuring 3 VLANs (VLAN10: Sales, VLAN20: HR, VLAN30: IT) on two Cisco 2960 switches, connecting via trunk connection with 9 PCs

### **2.1 Stage 1: Design**

<br>

<img width="1869" height="986" alt="изображение" src="https://github.com/user-attachments/assets/0d912390-049c-4150-8fb9-52175de4eec2" />

<br>

Here, PC7 is on **VLAN 10**, PC8 - **VLAN 20** and PC9 - **VLAN 30**

<br>

### **2.3 Physical Topology on Cisco Packet Tracer**



### **2.4 Create VLANs on a Second Switch**


```Cisco
SW2(config)#interface range f0/1-3

SW2(config-if-range)#switchport mode access
```

```Cisco
SW2(config)#vlan 10

SW2(config-vlan)#name HR

SW2(config-vlan)#vlan 20

SW2(config-vlan)#name SALES

SW2(config-vlan)#vlan 30

SW2(config-vlan)#name IT

SW2(config-vlan)#exit
```
<br>

<img width="471" height="41" alt="изображение" src="https://github.com/user-attachments/assets/c33dca81-3cac-47fa-9161-b70bff61d209" />

<br>

<img width="340" height="135" alt="изображение" src="https://github.com/user-attachments/assets/622c2c13-11bc-4fca-a10d-063259d98edf" />

<br>

##### Checking for created VLANs:

<br>

<img width="878" height="283" alt="изображение" src="https://github.com/user-attachments/assets/6206a28e-67af-4ff2-b344-c949392f5442" />

<br>

### **2.4 Assign VLANs to Newly Created PCs**

After we specified VLANs, we can  link them to the newly created PCs' interfaces

```Cisco
SW2(config)#int f0/1

SW2(config-if)#switchport access vlan 10

SW2(config-if)#int f0/2

SW2(config-if)#switchport access vlan 20

SW2(config-if)#int f0/3

SW2(config-if)#switchport access vlan 30
```

<br>

<img width="455" height="118" alt="изображение" src="https://github.com/user-attachments/assets/34f82a35-cbcb-4b9b-af93-593a74ab6296" />

<br>

```Cisco
SW2#show vlan brief
```

<br>

<img width="606" height="73" alt="изображение" src="https://github.com/user-attachments/assets/fb21c496-6ab5-4c52-8553-9523b67853c6" />


<br>

### **2.5 Configure Switch-to-Switch Trunk**

```Cisco
SW1>en

SW1#conf t

SW1(config)#int G0/1

SW1(config-if)#switchport mode trunk
```

<br>

<img width="411" height="123" alt="изображение" src="https://github.com/user-attachments/assets/cdb4a41b-2eaa-4dbf-892b-da6c9ea8e49b" />

<br>

This command sets Trunk Link between two switches


### **3. Veryfing**

To check which  interfaces  are in trunking mode, and what VLANs they are trunking, we can use command `show interaces trunk`
<br>

<img width="712" height="270" alt="изображение" src="https://github.com/user-attachments/assets/f0afca74-6b0b-4ac4-8ff2-6f446246ca54" />

<br>
From the image above, we can observe, that Gig0/1 carries VLANs 1 (default), 10, 20 and 30.


#### **Tagging in Practice**

If we run a simulation mode and ping PC7 (192.168.1.7) from PC1

<br>

<img width="622" height="417" alt="изображение" src="https://github.com/user-attachments/assets/2004d8ca-cbf8-45c5-ad4b-1a015d5bd3a7" />

<br>
<br>

The packets are transmitted successfuly (one packet failure is normal due to ARP Resolution process)


#### **Observing the Contents of DotQ1 packet**

If we observe a packet, that is transmitted from SW1 to SW2, we can notice an interesting detail

<br>
<br>

<img width="2401" height="1260" alt="изображение" src="https://github.com/user-attachments/assets/bd22600e-e322-41d1-ab4a-de928b7fbd34" />

<br>
<br>

<img width="786" height="260" alt="изображение" src="https://github.com/user-attachments/assets/024436f6-6926-49cb-ae0c-681ed9f5825c" />


<br>
<br>

<img width="650" height="116" alt="изображение" src="https://github.com/user-attachments/assets/f0cff23a-6b40-4927-89ca-091269411ba1" />

<br>
<br>

The Header type is Dot1q header, and observing its content, reveals us a new TCI block, which carries the VLAN ID

<br>
<br>

<img width="541" height="408" alt="изображение" src="https://github.com/user-attachments/assets/90a008ed-51a1-4638-b6ef-f0bb00039343" />

<br>
<br>
<br>
<img width="867" height="298" alt="изображение" src="https://github.com/user-attachments/assets/3b776136-f033-47ac-8290-00a3778a33d7" />


There is a **TCI (Tag Control Information)** field, which carries the information about VLAN ID

**It's structure is:**

- **PCP (Priority Control Point)** - 3 bits (0) - Defines Quality of Service 

- **DEI (Drop Eligible Indicator)** - 1 bit (x) indicates whether a frame can be dropped (0 = false, 1 = true); x = don't care

- **VID (VLAN ID)** - 12 bits (000a), which is written in hexadecimal and specifies the **ID of the VLAN the frame belongs to**

<br>

<img width="330" height="512" alt="изображение" src="https://github.com/user-attachments/assets/a361b8cc-4757-4e55-be82-6b7eadddbe87" />

<br>

Translating hexadecimal `a` to decimal, will give is **10**, which is indeed the VLAN ID of PC1 and PC7, that were communicating in the example above


### **Commands Used in the Write-Up**


| **Command**             | **Short Version** | **Executed in Mode**                    | **Purpose**                                                                                        |
| ----------------------- | ----------------- | --------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `switchport mode trunk` | `sw mo tr`        | Interface Config (`Switch(config-if)#`) | Changes the interface mode to **trunk**, which lets it to carry multiple VLAN ID using **tagging** |
