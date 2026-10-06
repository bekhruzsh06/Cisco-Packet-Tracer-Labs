# Inter-VLAN Routing - Router-on-a-Stick (ROAS)



## 1. Theory

**The Problem:** By definition, VLANs isolate traffic at layer 2. A PC in VLAN 10 cannot communicate with PC in VLAN 20 without a Layer 3 routing device, bridging the gap

**The Solution (ROAS):** Instead of using a separate physical cable for every VLAN's default gateway, ROAS uses a single trunk link, connected to a Router. The router's single physical interface is divided into muliple **virtual "sub-interfaces"**, one for each VLAN


![[ROAS(1).svg]]




## 2. Cisco Packet Tracer ROAS Set up

![[Pasted image 20261005000955.png]]


#### **1. Configure Trunk Link to a Router**

Switch port to router must be connected as a trunk, so it can handle tagged traffic

```Cisco
SW1(config)#int G0/2

SW1(config-if)#switchport mode trunk
```

![[Pasted image 20261005001433.png]]


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

![[Pasted image 20261005002007.png]]

**VLAN 20**

```Cisco
R1(config-if)#int G0/0/0.20
```

![[Pasted image 20261005002030.png]]


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

![[Pasted image 20261005002825.png]]


### **3. Checking**


`show ip interface brief` - Shows all interfaces with their IP addresses

![[Pasted image 20261005002942.png]]



#### **Pinging**

PC1 (VLAN 10) -> PC2 (VLAN 20) 

![[1005-ezgif.com-video-to-gif-converter.gif]]


Observing a SW1 -> R1 packet, gives us the familiar TCI `a`, which is 10, which stands for PC1 VLAN 10

![[Pasted image 20261005004908.png]]


However, if we observe a packet, that is returned by router to SW1, we may notice that the value of TCI has changed, and it's now `14`

![[Pasted image 20261005005012.png]]

We should keep in mind, that `14` is written in hex and if we convert it, we will get `20` in decimal, which is the ID of VLAN 20

(1 x (16^1)) + (4 x (16 ^ 0)) = 16 + (4 x 1) = 20


| **Command**                             | **Short Version**                   | **Executed in Mode**                           | **Purpose**                                                                                       |
| --------------------------------------- | ----------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `switchport mode trunk`                 | `sw mo tr`                          | Interface Config (`Switch(config-if)#`)        | Changes the interface mode to trunk, which lets it carry multiple VLAN IDs using tagging.         |
| `interface [type][port].[sub-id]`       | `int g0/0.10`                       | Global Config (`Router(config)#`)              | Creates and enters a virtual sub-interface on the router's physical port.                         |
| `encapsulation dot1Q [vlan_id]`         | `encap dot1q 10`                    | Sub-interface Config (`Router(config-subif)#`) | Instructs the router to tag/untag frames on this sub-interface with the specified 802.1Q VLAN ID. |
| `ip address [ip address] [subnet mask]` | `ip ad 192.168.1.254 255.255.255.0` | Interface Config (`Switch(config-if)#`)        | Assigns ip address and subnet mask to current interface                                           |
| `no shutdown`                           | `no shut`                           | Interface Config (`Router(config-if)#`)        | Powers on the physical interface so traffic can flow.                                             |
