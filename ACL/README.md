# ACL


**Access Control List (ACL)** is a sequential list of rules, applied to packets, going through router's interface. 

- Inspects packets and decide whether to **permit** or **deny** them (Gatekeeper)


While ACLs are the foundation of network security (firewalls), they are also used to identify traffic for other features, like Network Address Translation (NAT) or Quality of Service (QoS).

<br>
<br>

<img width="620" height="465" alt="изображение" src="https://github.com/user-attachments/assets/75126b69-01ce-460f-940f-71ae1e620925" />

<br>
<br>

### The Three Golden Rules of ACLs

1. **Top-Down Processing:** The router reads the rules in order (e.g., rule 10, then 20, then 30). The moment a packet matches a rule, the router takes action (permit or deny) and stops reading.

2. **The Implicit Deny:** At the bottom of every Cisco ACL is the rule, that says `deny all`. If a packet doesn't match a single rule, it will be **dropped** by default

3. **Inbound vs Outbound:** An ACL does nothing until applied to an interface in a specific direction
	- **Inbound(`in`):** Filters traffic **before** the router processes it
	- **Outbound (`out`):** Filters traffic **after** the router processes it, right as it exits the port.

## Stage 1: Design and Topology

We will use a single-router topology with two PCs on a private LAN attempting to reach a corporate server.

- **The LAN (Inside):** PC1 (`192.168.1.1`) and PC2 (`192.168.1.2`). Connected to R1's `G0/0/0` interface (`192.168.1.254`).
    
- **The Server Network (Outside):** Corporate Server (`10.0.0.10`). Connected to R1's `G0/1` interface (`10.0.0.1`).


<br>
<br>
<img width="1406" height="652" alt="изображение" src="https://github.com/user-attachments/assets/52329fd6-f474-4d1c-b8c6-0624a5c1ca41" />

<br>
<br>

<img width="1133" height="527" alt="изображение" src="https://github.com/user-attachments/assets/4c4e7b6c-b036-4868-969f-bc9c80322d61" />

<br>
<br>

## Stage 2: Standard ACL Configuration

**Standard ACLs (Numbered 1-9s9)** are simple. They can only filter traffic based on the **Source IP Address**.

- Because they don't know where the packet is actually going, best practice is to place them **as close to the destination as possible** so we don't accidentally block traffic meant for other networks.

**The Goal:** Block PC1 (Sales) from reaching the Server Network, but allow all other traffic.


### 1. Create the Standard ACL

We will create ACL 1. We use the keyword `host` to specify a single IP address (which is a shortcut for typing the `0.0.0.0` wildcard mask). We then explicitly permit everything else to override `deny all`.

```Cisco
R1(config)#access-list 1 deny host 192.168.1.1

R1(config)#access-list 1 permit any
```

<br>

<img width="531" height="64" alt="изображение" src="https://github.com/user-attachments/assets/5e3c455f-8018-4091-adbb-0c89cd065e9b" />
<br>
<br>

### 2. Apply the Standard ACL

Following the best practices, we place it close to the destination: R1's `G0/0/1` interface facing the server. We apply it on the outbound direction, meaning packets will be filtered as the leave the port.


```Cisco
R1(config)#int G0/0/1

R1(config-if)#ip access-group 1 out

R1(config-if)#exit
```


### 3. Verify

Ping from PC2 -> Server1: ✅Successful


<br>
<img width="691" height="388" alt="изображение" src="https://github.com/user-attachments/assets/f28511a6-464d-4b88-8c99-93738fbae2fd" />
<br>
<br>



Ping from PC1 -> Server1: ❌Destination host unreachable

<br>
<img width="633" height="397" alt="изображение" src="https://github.com/user-attachments/assets/72110983-f4c1-4bec-b6f3-aa4c5f021a0c" />
<br>
<br>



### Stage 3: Extended ACL Configuration


**Extended ACLs** (Numbered 100-199): Filter based on

- **Source IP** 
- **Destination IP**
- **Protocol** (TCP/UDP/ICMP)
- **Port Numbers** (80,443,22)


- Because they are so highly specific, best practice dictates placing them **as close to the source as possible** to kill bad traffic before it wastes bandwidth traveling across the network.


**The Goal:** We want to stop PC2 from browsing the web on the Corporate Server (HTTP / Port 80), but PC2 must still be able to Ping the server (ICMP) for troubleshooting.


(Note: First, remove the previous standard ACL so it doesn't interfere: `R1(config)# no access-list 1` and `R1(config-if)# no ip access-group 1 out` on G0/1).

### 1. Create the Extended ACL

We will create ACL 100. The syntax structure is: `access-list [number] [permit/deny] [protocol] [source] [destination] eq [port]


```Cisco
R1(config)# access-list 100 deny tcp 0.0.0.0 192.168.1.2 0.0.0.0 10.0.0.10 eq 80

R1(config)# access-list 100 permit ip any any
```
<br>
<br>
<img width="860" height="104" alt="изображение" src="https://github.com/user-attachments/assets/afa7f436-92ab-4441-a7e3-59b9d77618f1" />


-  Deny PC2 from establishing a TCP connection to the Server on port 80. Permit all other IP traffic from anyone to anywhere


### 2. Apply the Extended ACL

Following best practices, we place this close to the source: R1's `G0/0/0` interface facing the LAN. We apply it in the **inbound** direction, meaning "filter packets the moment they enter the router from the switch."

```Cisco
R1(config-if)# int G0/0/0

R1(config-if)# ip access-group 100 in

(config-if)# exit
```

<br>
<br>
<img width="657" height="107" alt="изображение" src="https://github.com/user-attachments/assets/f5abc311-cd10-45dd-a686-0bd78fd69a1a" />
<br>
<br>
<br>

**Verification:**

Pinging from PC2 -> Server 1: ✅Successful, because ICMP not denied

<img width="616" height="432" alt="изображение" src="https://github.com/user-attachments/assets/963804e3-b280-4179-aa8b-079af263e4f9" />
<br>
<br>


Opening web-page 10.0.0.10 from PC2: ❌Request Timeout, as HTTP to 10.0.0.10 is blocked

<br>
<img width="418" height="264" alt="изображение" src="https://github.com/user-attachments/assets/1ff62032-b673-4b71-833a-982068df5d67" />
<br>
<br>


### 3. Seeing All ACLs

`show access-lists` command lists all configured ACLs on the device

<br>
<img width="670" height="100" alt="изображение" src="https://github.com/user-attachments/assets/e791ab26-3f8b-427a-a26b-639a8ebf6233" />

<br>


### Commands Used in the Write-up

|**Command**|**Executed in Mode**|**Purpose**|
|---|---|---|
|`access-list [1-99] [permit/deny] [source-ip] [wildcard]`|Global Config|Creates a Standard ACL rule.|
|`access-list [100-199] [permit/deny] [protocol] [source] [destination] eq [port]`|Global Config|Creates an Extended ACL rule.|
|`ip access-group [number] [in/out]`|Interface Config (`Router(config-if)#`)|Applies an existing ACL to an interface in a specific direction.|
|`show access-lists`|Privileged EXEC (`Router#`)|Displays all ACLs configured on the router and how many packets have matched each rule.|
|`host [ip-address]`|_Within ACL Command_|A shorthand keyword for an exact IP match (replaces the `0.0.0.0` wildcard mask).|
|`any`|_Within ACL Command_|A shorthand keyword meaning "every IP address" (replaces the `255.255.255.255` wildcard mask).|
