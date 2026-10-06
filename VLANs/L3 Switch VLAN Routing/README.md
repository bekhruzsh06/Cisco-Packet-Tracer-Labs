# VLAN Routing Using L3 Switch

## **1. Theory**


##### **The Problems with ROAS** 

- While Router-on-a-Stick works well for a small number of VLANs, sending large number of inter-VLAN traffic through a single physical cable creates a bottleneck. 

- Routers process traffic in software, which is slower that a switch's hardware backplane


##### **Solution (Layer 3 Switching)**


- A Layer 3 switch combines **the hardware-switching speed** of a **switch** and **routing capabilities** of a **router**. 

- Instead of physical sub-interfaces on a router, we use **Switch Virtual Interfaces (SVI)** inside a switch to act as a default gateway 

**SVI:** Logical L3 switch interface that connects VLAN on the device to the L3 router engine on the same switch


<img width="1143" height="1197" alt="изображение" src="https://github.com/user-attachments/assets/7eb1fa48-a5b8-4d75-8a7a-3bea06f0ac46" />



**Setting up in Cisco Packet Tracer**

<br>
<img width="915" height="730" alt="изображение" src="https://github.com/user-attachments/assets/64e1c4d7-a672-48c9-8860-f681ee62387f" />
<br>

#### 2.1 Enable Global Routing

By default, Layer 3 switches act as dumb Layer 2 switches. You must explicitly tell the switch to act as a router.


```Cisco
L3SW1(config)#ip routing
```
<br>

#### 2.2 Set both G1/3 and G1/4 to trunk

As we have mixed VLANs on each side of L3 switch node, we have to set each node to trunk

```Cisco
L3SW1(config)#int G1/3

L3SW1(config-if)#switchport mode trunk

L3SW1(config)#int G1/4

L3SW1(config-if)#switchport mode trunk
```
<br>
<img width="846" height="340" alt="изображение" src="https://github.com/user-attachments/assets/e97a111d-ce7e-4a1c-b2c2-6fc3bc64ebc4" />
<br>

#### 2.3 Create VLANs and Assign Access Ports

Just like a standard switch, create the VLANs and assign the physical interfaces to them.


```Cisco
L3SW1(config)#vlan 10

L3SW1(config-vlan)#name HR

L3SW1(config-vlan)#vlan 20

L3SW1(config-vlan)#name SALES

L3SW1(config-vlan)#exit
```
<br>

<img width="404" height="130" alt="изображение" src="https://github.com/user-attachments/assets/c4749b0f-2833-40b5-a80c-670472e7bccd" />
<br>


#### 2.4 Create the Switch Virtual Interfaces (SVI)

Instead of going to a router to create gateways, you create a virtual interface for the VLAN directly to the switch


**VLAN 10 SVI**

```Cisco
L3SW1(config)#int vlan 10

L3SW1(config-if)#ip address 192.168.1.254 255.255.255.0

L3SW1(config-if)#exit
```
<br>
<img width="833" height="176" alt="изображение" src="https://github.com/user-attachments/assets/94e694ef-2a2b-483d-8495-30a439bf5b85" />
<br>
<br>

**VLAN 20 SVI**

```Cisco
L3SW1(config)#int vlan 20

L3SW1(config-if)#ip address 192.168.2.254 255.255.255.0

L3SW1(config-if)#exit
```
<br>
<img width="832" height="194" alt="изображение" src="https://github.com/user-attachments/assets/0e71f01b-8db3-46eb-8710-9faff1dd21b7" />
<br>
<br>


### 3. Checking
<br>
<img width="664" height="554" alt="изображение" src="https://github.com/user-attachments/assets/4777ddb6-c200-4c93-ac22-d6f06b1bd16e" />
<br>
<br>


Pinging from PC1->PC3 (Same VLAN and subnet) and PC4 (Different VLAN and subnet) => All successful 
