# Network Adress Translation (NAT and PAT)


Private IP addresses (like `192.168.x.x` or `10.x.x.x`) are legally not allowed to route across the public Internet. If a PC with a private IP tries to send data to a public web server, the web server will have no idea how to route the reply back to a private address.


**Network Address Translation (NAT)** solves this by
1. Intercepting packets as they leave local network
2. Stripping away the private Source IP
3. Replacing it with the router's public IP address


The most common implementation used in homes and enterprises is **PAT (Port Address Translation)**, commonly referred to in Cisco environments as **NAT Overload**. 

- It allows hundreds of internal devices to share a single public IP address by assigning a unique port number to each device's active session.

## 8.1 Stage 1: Design and Topology

For this lab, we will use a single edge router that sits on the boundary between a private local network and a public "Internet" server.

- **Inside Network (Private):** PC1 (`192.168.1.1`) and PC2 (`192.168.1.2`).
    
- **The NAT Router (R1):** Has a private LAN interface (`G0/0: 192.168.1.254`) and a public WAN interface (`G0/1: 203.0.113.1`).
    
- **Outside Network (Public):** A server acting as the Internet (`203.0.113.2`).

<br>
<img width="1255" height="674" alt="изображение" src="https://github.com/user-attachments/assets/33b648a4-d881-48bb-8fb7-fc8e689250d4" />
<br>
<br>

<img width="1156" height="499" alt="изображение" src="https://github.com/user-attachments/assets/849b0887-7c42-418a-8170-95a26183ed33" />
<br>

## Stage 2: Configuration

Configuring PAT requires three distinct steps:

1. Defining geographical borders of the network
2. Identifying the traffic using an ACL
3. Enabling the translation engine


#### 1. Define the Inside and Outside Interfaces

We must explicitly tell our router which port faces the private LAN and which to the public internet

```Cisco
R1(config)#int G0/0/0

R1(config-if)#ip nat inside



R1(config-if)#int G0/0/1

R1(config-if)#ip nat outside

R1(config-if)#exit
```

<br>
<img width="355" height="121" alt="изображение" src="https://github.com/user-attachments/assets/9b1baa20-3f45-493f-aac7-20d90d16eafc" />
<br>

#### 2. Create the Access Control List (ACL)

The NAT engine needs to know who is allowed to have their IP address translated. Create a standard ACL that targets our LAN `192.168.1.0/24`

We do not apply this ACL to an interface; we just create it so the NAT engine can reference it.

```Cisco
R1 (config) #access-list 1 permit 192.168.1.0 0.0.0.255
```

<br>
<img width="622" height="41" alt="изображение" src="https://github.com/user-attachments/assets/0200d118-1a71-4f92-aafb-1e5477c7b9d0" />
<br>
<br>

#### Configure NAT Overload

Now we put the pieces together. We tell the router: Take an internal traffic defined in the ACL list 1, push it out of the outside interface (G0/0/1), and overload it onto that single public IP address


```Cisco
R1(config)#ip nat inside source list 1 interface GigabitEthernet0/0/1 overload

R1(config)#exit
```

## Stage 3: Verification

Now, let's generate some traffic to see how PAT is working.

1. Ping from PC1 (`192.168.1.10`) to Server1 (`203.0.113.2`)

<br>
<img width="780" height="648" alt="изображение" src="https://github.com/user-attachments/assets/4dd47b24-c0bb-4913-aa86-8a6c06093114" />
<br>
<br>

2. Ping it again from PC2.
<br>
<img width="697" height="675" alt="изображение" src="https://github.com/user-attachments/assets/6d166d29-1eac-4b16-9a49-d7f2f93da9db" />

<br>
<br>
<br>
3. Go back to R1's CLI and view the NAT translations table:

```Cisco
show ip nat translations
```

<br>
<img width="816" height="350" alt="изображение" src="https://github.com/user-attachments/assets/a18fea71-9266-47b7-8ba0-a286e4014354" />
<br>

- **Inside local:** The true private IP addresses of PC1 and PC2

- **Inside global:** The public IP address, the server actually sees. Notice that R1 has masked both PCs with its own WAN IP (`203.0.113.1`), but assigned them unique port numbers (from `:16` to `28`) to keep their sessions cleanly separated.

| **Command**                                                                | **Executed in Mode**                    | **Purpose**                                                                                                  |
| -------------------------------------------------------------------------- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `ip nat inside`                                                            | Interface Config (`Router(config-if)#`) | Designates the interface connected to the private, internal network.                                         |
| `ip nat outside`                                                           | Interface Config (`Router(config-if)#`) | Designates the interface connected to the public, external network (Internet).                               |
| `ip nat inside source list [acl-number] interface [interface-id] overload` | Global Config (`Router(config)#`)       | Enables PAT. Translates the source IPs matching the ACL into the IP address of the specified exit interface. |
| `show ip nat translations`                                                 | Privileged EXEC (`Router#`)             | Displays the active NAT table, proving that private IPs are successfully masking as a public IP.             |
| `clear ip nat translation *`                                               | Privileged EXEC (`Router#`)             | Clears all dynamic NAT/PAT entries from the routing table, forcing the router to build fresh translations.   |
