# Inter-VLAN Routing - Router-on-a-Stick (ROAS)



## 1. Theory

**The Problem:** By definition, VLANs isolate traffic at layer 2. A PC in VLAN 10 cannot communicate with PC in VLAN 20 without a Layer 3 routing device, bridging the gap

**The Solution (ROAS):** Instead of using a separate physical cable for every VLAN's default gateway, ROAS uses a single trunk link, connected to a Router. The router's single physical interface is divided into muliple **virtual "sub-interfaces"**, one for each VLAN

<br>
<img width="1083" height="1079" alt="изображение" src="https://github.com/user-attachments/assets/56f507ea-5d32-4844-b9ed-ff4c78838866" />
<br>
<br>




## 2. Cisco Packet Tracer ROAS Set up

<br>
<img width="936" height="663" alt="изображение" src="https://github.com/user-attachments/assets/6a4517a6-ff9c-46f3-816e-474d784edc2d" />
<br>

#### **1. Configure Trunk Link to a Router**

Switch port to router must be connected as a trunk, so it can handle tagged traffic

```Cisco
SW1(config)#int G0/2

SW1(config-if)#switchport mode trunk
```

<br>
<img width="406" height="37" alt="изображение" src="https://github.com/user-attachments/assets/3fa72f7f-06cc-45ee-ad46-8bc0c65f7d01" />
<br>

#### **2. Configure Router's Sub Interfaces**

On the router, we activate the physical interface but **do not assign** it an **IP address**. Instead, we

1. Create virtual sub-interfaces
2. Assign 802.1Q encapsulation to match our existing VLAN IDs
3. Assign the gateway IP addresses


##### **2.1 Create Virtual Sub-Interfaces**

**VLAN 10**

```Cisco
R1(config-if)#int G0/0/0.10
```

<img width="980" height="120" alt="изображение" src="https://github.com/user-attachments/assets/d2cfe4ce-a16e-4e1d-969e-909652ed57ad" />

<br>
<br>

**VLAN 20**

```Cisco
R1(config-if)#int G0/0/0.20
```

<img width="972" height="106" alt="изображение" src="https://github.com/user-attachments/assets/caec07d1-5c4f-4edc-ac1f-592682d8f4fb" />
<br>
<br>


#### **2.2 Assign 802.Q encapsulation same as VLAN ID**

**VLAN 10**

```Cisco
R1(config-subif)#encapsulation dot1Q 10
```

**VLAN 20**

```Cisco
R1(config-subif)#encapsulation dot1Q 20
```

#### **2.3 Assign the Gateway IP Address**

**VLAN 10**

```Cisco
R1(config-subif)#ip address 192.168.1.254 255.255.255.0
```


**VLAN 20**

```Cisco
R1(config-subif)#ip address 192.168.2.254 255.255.255.0
```


**Complete Process**

<br>
<img width="990" height="369" alt="изображение" src="https://github.com/user-attachments/assets/7cb910e0-3ac4-4c91-affd-645bb5e48225" />
<br>

### **3. Checking**


`show ip interface brief` - Shows all interfaces with their IP addresses

<br>
<img width="812" height="160" alt="изображение" src="https://github.com/user-attachments/assets/859bce54-ed26-4e0b-80a4-033e91637c9b" />
<br>



#### **Pinging**

PC1 (VLAN 10) -> PC2 (VLAN 20) 

<br>
<img width="1280" height="976" alt="изображение" src="https://github.com/user-attachments/assets/cc1a7e3f-44f0-414b-b922-2e820b9da256" />



Observing a SW1 -> R1 packet, gives us the familiar TCI `a`, which is 10, which stands for PC1 VLAN 10

<img width="603" height="339" alt="изображение" src="https://github.com/user-attachments/assets/f3a68d7d-dfe2-407a-aad1-98f11c2e0540" />

<br>
<br>

However, if we observe a packet, that is returned by router to SW1, we may notice that the value of TCI has changed, and it's now `14`

<img width="592" height="653" alt="изображение" src="https://github.com/user-attachments/assets/aa82fc3f-42fe-41d0-9e45-2f0371d26b46" />
<br>
<br>

We should keep in mind, that `14` is written in hex and if we convert it, we will get `20` in decimal, which is the ID of VLAN 20

(1 x (16^1)) + (4 x (16 ^ 0)) = 16 + (4 x 1) = 20


| **Command**                             | **Short Version**                   | **Executed in Mode**                           | **Purpose**                                                                                       |
| --------------------------------------- | ----------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `switchport mode trunk`                 | `sw mo tr`                          | Interface Config (`Switch(config-if)#`)        | Changes the interface mode to trunk, which lets it carry multiple VLAN IDs using tagging.         |
| `interface [type][port].[sub-id]`       | `int g0/0.10`                       | Global Config (`Router(config)#`)              | Creates and enters a virtual sub-interface on the router's physical port.                         |
| `encapsulation dot1Q [vlan_id]`         | `encap dot1q 10`                    | Sub-interface Config (`Router(config-subif)#`) | Instructs the router to tag/untag frames on this sub-interface with the specified 802.1Q VLAN ID. |
| `ip address [ip address] [subnet mask]` | `ip ad 192.168.1.254 255.255.255.0` | Interface Config (`Switch(config-if)#`)        | Assigns ip address and subnet mask to current interface                                           |
| `no shutdown`                           | `no shut`                           | Interface Config (`Router(config-if)#`)        | Powers on the physical interface so traffic can flow.                                             |
