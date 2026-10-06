#VLAN Routing Using L3 Switch

## **1. Theory**


##### **The Problems with ROAS** 

- While Router-on-a-Stick works well for a small number of VLANs, sending large number of inter-VLAN traffic through a single physical cable creates a bottleneck. 

- Routers process traffic in software, which is slower that a switch's hardware backplane


##### **Solution (Layer 3 Switching)**


- A Layer 3 switch combines **the hardware-switching speed** of a **switch** and **routing capabilities** of a **router**. 

- Instead of physical sub-interfaces on a router, we use **Switch Virtual Interfaces (SVI)** inside a switch to act as a default gateway 

**SVI:** Logical L3 switch interface that connects VLAN on the device to the L3 router engine on the same switch


![[L3 Switch.svg]]




**Setting up in Cisco Packet Tracer**


![[Pasted image 20261005012654.png]]

#### 2.1 Enable Global Routing

By default, Layer 3 switches act as dumb Layer 2 switches. You must explicitly tell the switch to act as a router.


```Cisco
L3SW1(config)#ip routing
```

#### 2.2 Set both G1/3 and G1/4 to trunk

As we have mixed VLANs on each side of L3 switch node, we have to set each node to trunk

```Cisco
L3SW1(config)#int G1/3

L3SW1(config-if)#switchport mode trunk

L3SW1(config)#int G1/4

L3SW1(config-if)#switchport mode trunk
```

![[Pasted image 20261005014740.png]]

#### 2.3 Create VLANs and Assign Access Ports

Just like a standard switch, create the VLANs and assign the physical interfaces to them.


```Cisco
L3SW1(config)#vlan 10

L3SW1(config-vlan)#name HR

L3SW1(config-vlan)#vlan 20

L3SW1(config-vlan)#name SALES

L3SW1(config-vlan)#exit
```

![[Pasted image 20261005012923.png]]


#### 2.4 Create the Switch Virtual Interfaces (SVI)

Instead of going to a router to create gateways, you create a virtual interface for the VLAN directly to the switch


**VLAN 10 SVI**

```Cisco
L3SW1(config)#int vlan 10

L3SW1(config-if)#ip address 192.168.1.254 255.255.255.0

L3SW1(config-if)#exit
```

![[Pasted image 20261005014449.png]]

**VLAN 20 SVI**

```Cisco
L3SW1(config)#int vlan 20

L3SW1(config-if)#ip address 192.168.2.254 255.255.255.0

L3SW1(config-if)#exit
```

![[Pasted image 20261005014504.png]]




### 3. Checking

![[Pasted image 20261005014829.png]]


Pinging from PC1->PC3 (Same VLAN and subnet) and PC4 (Different VLAN and subnet) => All successful 
