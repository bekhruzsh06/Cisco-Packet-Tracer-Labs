
## **1. Theory**

**Static Routing:** Process of manually configuring a router's routing table to tell exactly how to reach to hosts in unknown remote networks

#### **Syntax:**

`ip route [Destination_Network_IP] [Destination_Network_Mask] [Next_Hop_IP]`


### Pros and Cons of Static Routing

- **Pros:** Perfect predictability, highly secure (no routing protocol advertisements to intercept), and uses zero extra CPU or bandwidth overhead. Excellent for small networks or "stub" networks (networks with only one way in or out).
    
- **Cons:** Does not adapt to topology changes. If a cable gets cut and a backup path exists, a static route will blindly keep sending traffic into the dead link. It is also an administrative nightmare to maintain manually on large enterprise networks.


## **2. Building in Cisco Packet Tracer**


### 1. Design

For this configuration, we will use a classic 2-router topology connecting two distinct sites.

- **Site A (Left):** Uses the `192.168.1.0/24` LAN. Connected to Router 1 (R1).
    
- **Site B (Right):** Uses the `192.168.2.0/24` LAN. Connected to Router 2 (R2).
    
- **The WAN Link:** Connects R1 and R2 using the `10.0.0.0/30` subnet.

<br>
<img width="1481" height="896" alt="изображение" src="https://github.com/user-attachments/assets/f286922a-a079-4a37-b269-f0b0d5f47dbc" />
<br>




### **1.1 Topology in Cisco Packet Tracer**


<br>
<img width="1291" height="732" alt="изображение" src="https://github.com/user-attachments/assets/9aef7518-758d-485e-a446-1a05ae77fd72" />
<br>



### **2. Configuring Process**


#### 1. Set up IP addresses for interfaces

Configure IP addresses on LAN and WAN interface on both routers

<br>

**R1:**

```Cisco
R1(config)#int G0/0/0

R1(config-if)#no shutdown

R1(config-if)#

R1(config-if)#ip address 192.168.1.254 255.255.255.0

R1(config-if)#exit

R1(config)#int G0/0/1

R1(config-if)#ip address 10.0.0.1 255.255.255.252

R1(config-if)#exit
```
<br>
<img width="894" height="636" alt="изображение" src="https://github.com/user-attachments/assets/e2e2921f-3891-491c-b189-5de156499516" />
<br>
<br>


**R2:**

```Cisco
R2(config)#int G0/0/0

R2(config-if)#no shutdown

R2(config-if)#ip address 192.168.2.254 255.255.255.0

R2(config-if)#exit

R2(config)#int G0/0/1

R2(config-if)#no shutdown

R2(config-if)#

R2(config-if)#ip address 10.0.0.2 255.255.255.252

R2(config-if)#exit

R2(config-if)#do show ip interfaces brief
```

<br>
<img width="943" height="713" alt="изображение" src="https://github.com/user-attachments/assets/83779a7c-e7e6-443e-8f11-426005c748a8" />
<br>
<br>


#### 2. Configure R1 (Routing to the Right)

R1 knows about `192.168.1.0` (its LAN) and `10.0.0.0` (its WAN link). It must be manually taught how to reach R2's LAN.

```Cisco
R1(config)#ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

- **Meaning:** To reach the 192.168.2.0, send packet to 10.0.0.2 host


#### 3. Configure R2 (Return Path)

**Crucial Rule:** Routing is a two-way process. If you only configure R1, PC1's ping will successfully reach PC2. However, when PC2 tries to send the "Ping Reply" back, R2 will look at its routing table, realize it has no idea where `192.168.1.0` is, and drop the packet. Therefore, we must configure the reverse path on R2.

```Cisco
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

- **Meaning:** To reach the 192.168.1.0, send packet to 10.0.0.1 host

#### 4 Verifying Static Routes

To verify that the router accepted your configuration, check the routing table.

```Cisco
R1# show ip route
```
<br>
<img width="880" height="383" alt="изображение" src="https://github.com/user-attachments/assets/317b913a-551c-49e0-85a9-13c85f33cbf6" />
<br>

```Cisco
R2# show ip route
```
<br>
<img width="834" height="536" alt="изображение" src="https://github.com/user-attachments/assets/dcdbb807-5a52-49f1-be81-9f14d964a595" />
<br>

- **S (Static):** Route, that was configured by us manually
- **C (Connected):** Routes, to which router is connected physically
- **L (Local):** Local address of the device

#### **Static Route Structure Breakdown**

```Cisco
192.168.1.0/24 [1/0] via 10.0.0.1
```

- `192.168.1.0/24` - Destination network

- `[1/0]` - values, representing Administrative Distance and Metric
	-  **Administrative Distance (1):** trustworthiness of the path (The less is value, the more authority it has) 
	- **Metric (0):** Cost of specific path (hop count, bandwidth, delay)

- `10.0.0.1` Next IP address (hop) to send packet to reach the intended network


| **Command**                               | **Short Version** | **Executed in Mode**              | **Purpose**                                                                                         |
| ----------------------------------------- | ----------------- | --------------------------------- | --------------------------------------------------------------------------------------------------- |
| `ip route [network] [mask] [next-hop-ip]` | `ip route ...`    | Global Config (`Router(config)#`) | Creates a static route to a remote network, directing traffic to a neighboring router's IP address. |
| `show ip route`                           | `sh ip route`     | Privileged EXEC (`Router#`)       | Displays the routing table. Look for the 'S' code to verify static routes are active.               |
