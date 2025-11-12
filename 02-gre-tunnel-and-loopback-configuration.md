## **2 GRE Tunnel and Loopback Configuration**

## **2.1 Introduction**

This chapter focuses on establishing a GRE tunnel between Branch-A and Branch-B through the ISP network. The tunnel creates a secure logical link allowing internal LANs from both branches to communicate as if they were part of the same extended network. Loopback interfaces are configured on each router to serve as stable virtual endpoints for testing. Static routes (ip route) are also implemented to enable reachability between LAN and loopback networks across the GRE tunnel.


![](images/Pasted%20image%2020251112173156.png)


## **2.2 Topology**

| Device        | Interface(s)                     | Network / Connection                                                                        | IP Address                     | Subnet Mask                        | Gateway                      | Purpose                                                                |
| ------------- | -------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------ | ---------------------------------- | ---------------------------- | ---------------------------------------------------------------------- |
| **R1**        | Gi0/0<br>Gi0/1                   | LAN Branch-A (SW1)<br>WAN to ISP                                                            | 192.168.10.1<br>100.64.1.1     | 255.255.255.0<br>255.255.255.252   | N/A                          | LAN gateway<br>WAN uplink                                              |
| **ISP**       | Gi0/1<br>Gi0/2                   | Link to R1 <br>Link to R2                                                                   | 100.64.1.2<br>100.64.2.1       | 255.255.255.252<br>255.255.255.252 | N/A                          | Transit to R2 <br>Transit to R1                                        |
| **R2**        | Gi0/2<br>Gi0/0                   | WAN to ISP<br>LAN Branch-B (SW3)                                                            | 100.64.2.2<br>192.168.20.1     | 255.255.255.252<br>255.255.255.0   | N/A                          | WAN uplink<br>LAN gateway                                              |
| **SW1**       | Gi0/1<br>Gi0/0<br>Gi0/2<br>Gi0/3 | **→ R1** (uplink)**<br>→ PC1** (access)<br>**→ SW2** (STP link A)<br>**→ SW2** (STP link B) | N/A                            | N/A                                | N/A                          | Uplink to R1<br>Access for PC1<br>Redundancy (STP)<br>Redundancy (STP) |
| **SW2**       | Gi0/2<br>Gi0/3<br>Gi0/0          | **→ SW1** (STP link A)<br>**→ SW1** (STP link B)<br>**→ PC2** (access)                      | N/A                            | N/A                                | N/A                          | Redundancy (STP)<br>Redundancy (STP)<br>Access for PC2                 |
| **SW3**       | Gi0/3<br>Gi0/0                   | **→ R2** (uplink)<br>**→ PC3** (access)                                                     | N/A                            | N/A                                | N/A                          | Uplink to R2<br>Access for PC3                                         |
| **PC1 / PC2** | e0<br>e0                         | Branch-A LAN (via SW1)<br>Branch-A LAN (via SW2)                                            | 192.168.10.10<br>192.168.10.20 | 255.255.255.0<br>255.255.255.0     | 192.168.10.1<br>192.168.10.1 | End host<br>End host                                                   |
| **PC3**       | e0                               | Branch-B LAN (via SW3)                                                                      | 192.168.20.10                  | 255.255.255.0                      | 192.168.20.1                 | End host                                                               |



## 2.3 **Steps**

1. Configure loopback interfaces on both branch routers to create stable virtual endpoints for tunnel testing.
    
2. Establish a GRE tunnel between R1 and R2 using their WAN interface IP addresses as tunnel source and destination.
    
3. Assign IP addresses to the GRE tunnel interfaces to form a logical point-to-point link.
    
4. Configure static routes on both routers to enable communication between LAN networks and loopback interfaces across the GRE tunnel.
    
5. Verify tunnel status, routing table entries, and connectivity using diagnostic commands and ping tests between branch networks.



## **2.4 Loopback Interfaces Configuration**

Loopback interfaces are configured on both branch routers to provide stable, logical IP addresses that are always active regardless of physical link states. These interfaces serve as reliable test points for GRE tunnel verification and end-to-end connectivity checks. Each loopback interface uses a /32 mask, representing a single host address for diagnostic purposes.

|Device|Interface|IP Address|Subnet Mask|Purpose|
|---|---|---|---|---|
|R1|Lo0|10.1.1.1|255.255.255.255|Logical interface for GRE ping test|
|R2|Lo0|10.2.2.2|255.255.255.255|Logical interface for GRE ping test|

#### Router R1 – Loopback Configuration

```
enable
configure terminal
interface loopback0
ip address 10.1.1.1 255.255.255.255
no shutdown
end
write memory
```
![](images/Pasted%20image%2020251112151729.png)

#### Router R2 – Loopback Configuration

```
enable
configure terminal
interface loopback0
ip address 10.2.2.2 255.255.255.255
no shutdown
end
write memory
```
![](images/Pasted%20image%2020251112151946.png)


> **Notes.:** The configured loopback interfaces serve as logical and stable test points for GRE tunnel verification. They are not used as tunnel source or destination in this setup and do not act as backup links. GRE tunnels currently rely on the WAN physical interfaces for connectivity.



## 2.5 GRE Tunnel Configuration

The GRE tunnel provides a logical connection between Branch-A (R1) and Branch-B (R2) through the ISP network. It allows both LANs to communicate over the WAN as if they were directly connected. The tunnel uses the WAN IP addresses of both routers as source and destination endpoints.

|Device|Tunnel Interface|IP Address|Source IP|Destination IP|Description|
|---|---|---|---|---|---|
|R1|Tunnel0|172.16.0.1|100.64.1.1|100.64.2.2|GRE tunnel from R1 to R2 (over ISP)|
|R2|Tunnel0|172.16.0.2|100.64.2.2|100.64.1.1|GRE tunnel from R2 to R1 (over ISP)|

#### Router R1 – GRE Tunnel Configuration

```
enable
configure terminal
interface Tunnel0
ip address 172.16.0.1 255.255.255.252
tunnel source 100.64.1.1
tunnel destination 100.64.2.2
no shutdown
exit
write memory
```
![](images/Pasted%20image%2020251112152729.png)

#### Router R2 – GRE Tunnel Configuration

```
enable
configure terminal
interface Tunnel0
ip address 172.16.0.2 255.255.255.252
tunnel source 100.64.2.2
tunnel destination 100.64.1.1
no shutdown
exit
write memory
```
![](images/Pasted%20image%2020251112152938.png)



> **Notes.:** The GRE tunnel uses the WAN physical IP addresses as the source and destination. It forms a logical point-to-point link between R1 and R2 through the ISP network, allowing inter-branch communication independent of the physical topology.



## **2.6 Static Routes Configuration**

Static routes are configured on both branch routers to establish full reachability between all networks: LANs, loopbacks, and GRE tunnel interfaces. Each router learns about the remote networks and uses the GRE tunnel as the forwarding path.

|Device|Destination Network|Subnet Mask|Next Hop|Description|
|---|---|---|---|---|
|R1|192.168.20.0|255.255.255.0|172.16.0.2|Branch-B LAN through GRE tunnel|
|R1|10.2.2.2|255.255.255.255|172.16.0.2|R2 loopback via GRE tunnel|
|R2|192.168.10.0|255.255.255.0|172.16.0.1|Branch-A LAN through GRE tunnel|
|R2|10.1.1.1|255.255.255.255|172.16.0.1|R1 loopback via GRE tunnel|

#### Router R1 – Static Route Configuration

```
enable
configure terminal
ip route 192.168.20.0 255.255.255.0 172.16.0.2
ip route 10.2.2.2 255.255.255.255 172.16.0.2
exit
write memory
```
![](images/Pasted%20image%2020251112160112.png)
#### Router R2 – Static Route Configuration

```
enable
configure terminal
ip route 192.168.10.0 255.255.255.0 172.16.0.1
ip route 10.1.1.1 255.255.255.255 172.16.0.1
exit
write memory
```
![](images/Pasted%20image%2020251112160413.png)

## **2.7 Troubleshooting GRE Connectivity**

When GRE tunnels fail to establish between R1 and R2, the issue often lies in missing Layer 3 reachability between the GRE endpoints (the WAN IP addresses of both routers). Each branch router only knows its directly connected WAN subnet and cannot reach the remote GRE destination IP through the ISP without explicit routing.

In this topology, R1 belongs to the 100.64.1.0/30 network and R2 belongs to the 100.64.2.0/30 network. The ISP connects these two subnets, but both branch routers must know how to reach the opposite WAN IP through the ISP in order to form the GRE tunnel.

#### Fix – Add Host Routes for GRE Endpoints

**On Router R1:**

```
enable
configure terminal
ip route 100.64.2.2 255.255.255.255 100.64.1.2
end
write memory
```
![](images/Pasted%20image%2020251112163842.png)

**On Router R2:**

```
enable
configure terminal
ip route 100.64.1.1 255.255.255.255 100.64.2.1
end
write memory
```
![](images/Pasted%20image%2020251112163947.png)

These host routes allow each router to reach the remote GRE destination via the ISP, enabling successful tunnel formation and restoring full inter-branch connectivity.



> **Notes.:** Host routes (/32) are used for the GRE endpoint IPs because each router must reach only a single remote11 address — the WAN IP of the opposite router. Using a full network route (/30) is unnecessary, as the ISP already connects both subnets. In contrast, LAN and loopback routes use their full subnet masks to cover entire networks.



## **2.8 GRE and Loopback Diagnostics**

This section verifies full network functionality between branches. It includes loopback connectivity, GRE tunnel operation, and end-to-end communication tests. All pings are successful, confirming that static routes and GRE configurations are operating correctly.

#### **Router R1 – Connectivity Tests**

- Verify reachability to R2 loopback interface.
    

```
ping 10.2.2.2
```
![](images/Pasted%20image%2020251112165844.png)

- Verify reachability through GRE tunnel interface.
    

```
ping 172.16.0.2
```
![](images/Pasted%20image%2020251112165905.png)

**Result:** Both pings are successful, confirming proper route advertisement and tunnel functionality.

#### **Router R2 – Connectivity Tests**

- Verify reachability to R1 loopback interface.
    

```
ping 10.1.1.1
```
![](images/Pasted%20image%2020251112165935.png)

- Verify reachability through GRE tunnel interface.
    

```
ping 172.16.0.1
```
![](images/Pasted%20image%2020251112165953.png)

#### **Result** 

Both pings are successful, confirming symmetric connectivity across the tunnel.



## 2.9 **Interface and Tunnel Verification**

#### Router R1 – Interface Status and Packet Movement

- Verify operational state of all interfaces.
    

```
show ip interface brief
```
![](images/Pasted%20image%2020251112170230.png)

- Display detailed GRE tunnel information and packet flow statistics.
    

```
show interfaces tunnel0
```
![](images/Pasted%20image%2020251112170256.png)

- Display packet counters and transmission statistics for Loopback0.
    

```
show interfaces loopback0
```
![](images/Pasted%20image%2020251112170612.png)
#### Router R2 – Interface Status and Packet Movement

- Verify operational state of all interfaces.
    

```
show ip interface brief
```
![](images/Pasted%20image%2020251112170020.png)

- Display detailed GRE tunnel information and packet flow statistics.
    

```
show interfaces tunnel0
```
![](images/Pasted%20image%2020251112170039.png)

- Display packet counters and transmission statistics for Loopback0.
    

```
show interfaces loopback0
```
![](images/Pasted%20image%2020251112170628.png)

#### **Summary** 

All interfaces (Loopback and GRE Tunnel) report as up/up, with packet counters confirming active data transmission between R1 and R2.

---

### Ping Between PCs

- Verify full end-to-end communication from Branch A to Branch B.
    

**From PC1 to PC3:**

```
ping 192.168.20.10
```
![](images/Pasted%20image%2020251112170657.png)

**From PC3 to PC1:**

```
ping 192.168.10.10
```
![](images/Pasted%20image%2020251112170744.png)

#### **Result**

Both ping tests are successful, confirming complete network connectivity between branches


## 2.10 **Conclusion**

This chapter establishes a logical GRE connection between both branch routers through the ISP network. Loopback interfaces are configured as stable test points, ensuring reliable verification of inter-router communication. IP routes provide full reachability between LANs, loopbacks, and tunnel interfaces. After successful troubleshooting and verification, the network confirms symmetric connectivity and data flow through the GRE tunnel, enabling end-to-end communication between all branch devices.

---

Next part:




















































































































