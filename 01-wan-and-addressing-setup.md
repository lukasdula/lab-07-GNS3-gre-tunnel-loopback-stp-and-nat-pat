
# **1 WAN and Addressing Setup**

## **1.1 Introduction**

This chapter defines the IP addressing for all core network devices, including routers, the ISP node, and end hosts. It establishes the basic communication backbone between both branches and the service provider. Each router receives WAN and LAN interfaces, while switches and PCs are connected to their respective segments. The addressing plan ensures clear separation between local and transit networks, preparing the foundation for the following GRE tunnel and loopback configuration.


![](images/Pasted%20image%2020251112144036.png)


## **1.2 Topology**

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


## **1.3 Steps**

1. Assign IP addresses to all PCs according to the addressing table.
    
2. Configure IP addresses on R1 interfaces (LAN and WAN to ISP).
    
3. Configure IP addresses on R2 interfaces (LAN and WAN to ISP).
    
4. Set IP addresses on ISP router interfaces linking R1 and R2.
    
5. Perform basic ping and show-command diagnostics between key devices (PCs, routers, and WAN links).





## **1.4 PCs - IP Address Configuration**

This section defines the IP addressing for all end devices in both branches. Each PC receives a unique IP address, subnet mask, and default gateway according to the addressing plan. These parameters ensure proper communication within the local LAN before tunnel and WAN testing.

### **PC1**

- Configure IP address, subnet mask, and default gateway.
    

```
IP Address: 192.168.10.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
show ip
save
```
![](images/Pasted%20image%2020251112021935.png)
### **PC2**

- Configure IP address, subnet mask, and default gateway.
    

```
IP Address: 192.168.10.20
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.10.1
show ip
save
```
![](images/Pasted%20image%2020251112022208.png)
### **PC3**

- Configure IP address, subnet mask, and default gateway.
    

```
IP Address: 192.168.20.10
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.20.1
show ip
save
```
![](images/Pasted%20image%2020251112022320.png)


> **Notes.:** PC1 and PC2 share the same default gateway because they belong to the same LAN network (192.168.10.0/24).  
> Even though they are connected to different switches (SW1 and SW2), both switches operate within a single Layer 2 domain.  
> The gateway 192.168.10.1 is assigned to interface Gi0/0 on router R1, which provides Layer 3 routing from the LAN to other networks.  
> Different gateways would be required only if PC1 and PC2 were placed in separate VLANs, meaning different IP subnets.

## **1.5 Router (R1) - IP Address Configuration**

This section assigns IP addresses to R1 interfaces. The LAN interface provides connectivity for Branch-A internal devices, while the WAN interface connects to the ISP network. This configuration forms the base for inter-branch communication.

#### Commands for G0/0 (LAN – toward Switch SW1)

```
enable
configure terminal
interface Gi0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
write memory
```
![](images/Pasted%20image%2020251112024156.png)

#### Commands for G0/1 (WAN – toward ISP Router)

```
enable
configure terminal
interface Gi0/1
ip address 100.64.1.1 255.255.255.252
no shutdown
end
write memory
```
![](images/Pasted%20image%2020251112024355.png)

## **1.6 Router (R2) - IP Address Configuration**

This section assigns IP addresses to R2 interfaces. The LAN interface provides connectivity for Branch-B internal devices, while the WAN interface connects to the ISP network. This configuration completes the addressing setup for both branches, allowing interconnection through the ISP.

#### Commands for Gi0/0 (LAN – toward Switch SW3)

```
enable
configure terminal
interface Gi0/0
ip address 192.168.20.1 255.255.255.0
no shutdown
end
write memory
```
![](images/Pasted%20image%2020251112025116.png)
#### Commands for Gi0/2 (WAN – toward ISP Router)

```
enable
configure terminal
interface Gi0/2
ip address 100.64.2.2 255.255.255.252
no shutdown
end
write memory
```
![](images/Pasted%20image%2020251112025223.png)

## **1.7 Router (ISP) - IP Address Configuration**

This section assigns IP addresses to the ISP router interfaces. These interfaces create the WAN transit connections between Branch-A (R1) and Branch-B (R2), allowing communication through the service provider’s network. This configuration finalizes the WAN addressing setup.

#### Commands for Gi0/1 (Link to R1)

```
enable
configure terminal
interface Gi0/1
ip address 100.64.1.2 255.255.255.252
no shutdown
exit
write memory
```
![](images/Pasted%20image%2020251112025808.png)
#### Commands for Gi0/2 (Link to R2)

```
enable
configure terminal
interface Gi0/2
ip address 100.64.2.1 255.255.255.252
no shutdown
exit
write memory
```
![](images/Pasted%20image%2020251112025906.png)



## **1.8 Network Diagnostics and Connectivity Tests**

This section verifies the basic IP addressing and connectivity between key network devices. The goal is to confirm that interfaces are correctly configured and that essential links between LAN and WAN segments are operational before GRE tunnel and STP implementation.

#### **Router R1 – Interface Verification**

- Check the status and assigned IP addresses of all interfaces.
    

```
show ip interface brief
```
![](images/Pasted%20image%2020251112031112.png)
#### **Router R1 – Ping Tests**

- Verify connectivity from R1 to PC1 (LAN segment).
    

```
ping 192.168.10.10
```
![](images/Pasted%20image%2020251112031138.png)
- Verify connectivity from R1 to ISP (WAN segment).
    

```
ping 100.64.1.2
```

#### **PC3 – Connectivity Test**

- Test communication from PC3 to its default gateway on router R2.
    

```
ping 192.168.20.1
```
![](images/Pasted%20image%2020251112031234.png)

#### **PC2 – Connectivity Test**

- Test communication from PC2 to its default gateway on router R1.
    
 ```
 ping 192.168.10.1
 ```
![](images/Pasted%20image%2020251112031748.png)

> **Notes.:** Ping from PC2 to the gateway (192.168.10.1) is also successful. Spanning Tree Protocol (STP) is not yet active, so both links between SW1 and SW2 remain operational until configured in the next section.


## 1.9 **Conclusion**

The WAN and addressing configuration establishes a functional IP structure between both branches and the ISP network. All local devices successfully communicate with their default gateways, confirming proper LAN setup. Router and ISP interfaces are fully operational, providing a stable foundation for implementing **static routing (ip route)**, **GRE tunneling**, and **loopback interfaces** in the upcoming chapters.

---

**Next part:** 
















































