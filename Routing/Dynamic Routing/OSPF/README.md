# Dynamic Routing OSPF (Open Shortest Path First)

## **1. Theory**

While **Static Routing** works well for small, predictable networks, it quickly becomes unmanageable as an enterprise grows. If a single cable breaks in a static environment, traffic stops completely unless an administrator manually rewrites the routing table.

**Dynamic Routing Protocols** solve this by allowing routers to talk with each other, discover remote networks and calculate the best path in real-time

**OSPF (Open Shortest Path First)** is the most widely used interior dynamic routing protocol in enterpise networks


### How OSPF Works (Link-State Concept)

Unlike distance-vector protocols (like RIP) that simply ask neighboring routers for a list of known destinations, OSPF functions like a detailed GPS map:

1. **Neighbor Discovery:** Routers send out special `Hello` packets to find adjacent routers on directly connected links

2. **Database Synchronization:** Once neighbours are established, they exchange state information about their links (speeds, statuses, connections). Every router builds an identical, complete topology map of the **entire network** (the Link-State Database)

3. **The SPF Alogirthm:** Using Dijkstra's Algorithm, each router independently calculates the absolute shortest path to every network, building its own routing table.

4. **Adaptability:** If a link goes down, routers instantly detect the change, recalculate a new path via the algorithm, and update their routing tables automatically with zero manual intervention.

## **2. Implementation**

### 1: Design

For this OSPF configuration, we will scale up to a **3-Router Linear Topology** to properly demonstrate dynamic discovery:

<br>
<img width="988" height="1229" alt="изображение" src="https://github.com/user-attachments/assets/adbcd7b4-0a06-4407-b8f5-4a7798b697a0" />
<br>

### **CPT Build**


First of all, we need to add serial and Gigabit modules to our routers (NIM-2T and NIM-ES2-4)

<br>

<img width="1304" height="428" alt="изображение" src="https://github.com/user-attachments/assets/8b527522-ffe6-4293-8b38-3edb6ea68eae" />

<br>

Following that, our topology is as follows:

<br>

<img width="1333" height="1140" alt="изображение" src="https://github.com/user-attachments/assets/8dbe2c06-5e74-4650-a585-4b3d306e01d4" />

<br>

#### 2.1  Understanding Wildcard Masks

When configuring OSPF, you must tell the router which interfaces to activate using a **Wildcard Mask**. A wildcard mask is the exact inverse of a standard subnet mask:

- Standard Subnet Mask for `/24`: `255.255.255.0`
    
- OSPF Wildcard Mask for `/24`: `0.0.0.255` _(Zeros mean "match this exact part of the IP", ones mean "ignore/any")_
    
- Subnet Mask for a point-to-point `/30` WAN link: `255.255.255.252`
    
- OSPF Wildcard Mask for `/30`: `0.0.0.3`



#### 2.2 Configuring OSPF on R1, R2, and R3


To run OSPF, you assign a **Process ID** (1-65535, e.g `1`) and assign your network to an area (for basic setups, all networks belong to the backbone, **Area 0**).

**Area** - Space or room, where specified networks with masks are stored, so they can interact with each and exchange routing data

**Analogy:** Imagine, you are a Head of Department. Instead of explicitly telling each worker what to do (`static routing`), you place them in one room (`area`), so they can exchange their ideas and thoughts (`dynamic routing`)


#### Configuring Router 1 (Site A)

Tell R1 to advertise its LAN (`192.168.1.0`) and its connection to the R2 WAN link (`10.0.0.0`).

```Cisco
R1> enable

R1# configure terminal

R1(config)# router ospf 1

R1(config-router)# network 192.168.1.0 0.0.0.255 area 0

R1(config-router)# network 10.0.0.0 0.0.0.3 area 0

R1(config-router)# exit
```
<br>

<img width="666" height="95" alt="изображение" src="https://github.com/user-attachments/assets/5a881c8d-d664-4353-94f6-f752f4d5317d" />

<br>

 So now we have networks 192.168.1.0/24 and 10.0.0.0/30 in the same room, so they can exchange their paths
 <br>
#### Configuring Router 2 (Site B - The Core)

R2 sits in the middle and must advertise its own LAN plus _both_ of its WAN connections.


```Cisco
R2> enable

R2# configure terminal

R2(config)# router ospf 1

R2(config-router)# network 192.168.2.0 0.0.0.255 area 0

R2(config-router)# network 10.0.0.0 0.0.0.3 area 0

R2(config-router)# network 10.0.4.0 0.0.0.3 area 0

R2(config-router)# exit
```
<br>
<img width="898" height="288" alt="изображение" src="https://github.com/user-attachments/assets/4f0a2daa-6975-45a5-9a2c-4f0d04c3f3df" />
<br>


#### Configuring Router 3 (Site C)

Tell R3 to advertise its LAN and its connection to the R2 WAN link.


```Cisco
R3> enable

R3# configure terminal

R3(config)# router ospf 1

R3(config-router)# network 192.168.3.0 0.0.0.255 area 0

R3(config-router)# network 10.0.4.0 0.0.0.3 area 0

R3(config-router)# exit
```

<br>
<img width="646" height="102" alt="изображение" src="https://github.com/user-attachments/assets/e7cff27d-748f-4e20-9522-338c6233c9af" />
<br>

###  Verifying OSPF Operation

Once OSPF is configured on all routers, they will send out Hello packets, discover each other, and exchange routing tables automatically within a few seconds.

### 1. Check OSPF Neighbors

Verify that your routers have successfully formed an adjacency with their neighbors:

```Cisco
R1# show ip ospf neighbor
```


### 2. Check the Routing Table

Examine the routing table to see dynamically learned routes:



```Cisco
R1# show ip route
```

**R1**

<br>
<img width="721" height="255" alt="изображение" src="https://github.com/user-attachments/assets/3eeacdf3-b23c-446e-bff9-4dc59854eec3" />

<br>


**R2**
<br>
<img width="707" height="222" alt="изображение" src="https://github.com/user-attachments/assets/008aad86-e03b-4a7d-9a89-8ecbab9a5419" />

<br>


**R3**
<br>
<img width="750" height="234" alt="изображение" src="https://github.com/user-attachments/assets/8141c65f-f310-4bf6-966d-7d1083d568b5" />


<br>

Look for lines starting with an **`O`** (which stands for OSPF).

We will see that R1 automatically knows how to reach R3's network (`192.168.3.0`) via R2 and vice versa, even though we've never wrote a static route for it!


#### **Pinging**

**PC1** (192.168.1.1) -> **PC3** (192.168.3.1) ✅

<br>

![Uploading изображение.png…]()

<br>

## OSPF Commands Summary

|**Command**|**Short Version**|**Executed in Mode**|**Purpose**|
|---|---|---|---|
|`router ospf [process-id]`|`router ospf 1`|Global Config (`Router(config)#`)|Enables the OSPF routing process and enters OSPF configuration mode.|
|`network [ip] [wildcard] area [id]`|`net 192.168.1.0 0.0.0.255 ar 0`|OSPF Config (`Router(config-router)#`)|Tells OSPF which local interfaces/subnets to participate in routing and assign them to an Area.|
|`show ip ospf neighbor`|`sh ip ospf neigh`|Privileged EXEC (`Router#`)|Displays active neighbor relationships and state handshakes.|
|`show ip route`|`sh ip route`|Privileged EXEC (`Router#`)|Displays the routing table, highlighting dynamically learned OSPF paths with an `O` code.|
