# **4 - NAT and PAT Configuration and Verification**


## **4.1 Introduction**

Network Address Translation (NAT) and Port Address Translation (PAT) are used to allow private IP addresses from internal networks to access external networks such as the internet. NAT converts private IP addresses into public ones, while PAT extends this by allowing multiple internal devices to share a single public address through different port numbers. In this demonstration, NAT and PAT are configured only on router **R1** for Branch-A to show the translation process between the internal LAN and the ISP network. The GRE tunnel and Branch-B router (R2) remain unaffected by this configuration.

![](images/Pasted%20image%2020251112201555.png)

## **4.2 Topology**

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



## 4.3 **Steps**


1. Configure inside and outside interfaces on R1.
    
2. Define an access list on R1 for local addresses.
    
3. Configure PAT (overload) on R1 using the WAN interface.
    
4. Perform diagnostic verification and testing of NAT and PAT operation.
    


## **4.4 NAT and PAT Configuration and Verification**

**Configure inside and outside interfaces on R1**

The router R1 defines its LAN-facing and WAN-facing interfaces to establish which interfaces participate in NAT. The LAN interface is marked as **inside**, and the WAN interface toward the ISP is marked as **outside**. This distinction allows the router to know where translation should happen.

**Configuration summary (R1)**

- Interface GigabitEthernet0/0 is configured as the inside interface.
    
- Interface GigabitEthernet0/1 is configured as the outside interface.
    

#### **Command in CLI:**

```
enable
configure terminal
interface Gi0/0
ip nat inside
exit
interface Gi0/1
ip nat outside
end
```
![](images/Pasted%20image%2020251112194654.png)

### **Verification**


 ```
show running-config | include nat
 ```
![](images/Pasted%20image%2020251112194738.png)
#### **Results**

Displays lines in the running configuration that contain NAT commands. Useful to confirm that interfaces are properly marked as inside or outside.


## **4.5 Define an access list on R1 for local addresses**

The router R1 defines a standard access list to identify the range of internal IP addresses that will be translated when communicating with external networks. This access list allows all devices from the LAN subnet 192.168.10.0/24 to use NAT and PAT for outbound connections.

**Configuration summary (R1)**

- The access list is used to match all local IP addresses from the 192.168.10.0/24 network.
    
- It will later be linked to the NAT overload configuration.
    

#### **Command in CLI:**

```
enable
configure terminal
access-list 1 permit 192.168.10.0 0.0.0.255
end
```
![](images/Pasted%20image%2020251112195751.png)

**Verification**

```
show access-lists
```
![](images/Pasted%20image%2020251112195810.png)

Displays the configured access lists and the number of matches for each entry, confirming that ACL 1 exists and is correctly defined for the LAN subnet.

```
show running-config | include access-list
```
![](images/Pasted%20image%2020251112195856.png)

Verifies that the access list statement is present in the configuration and ready to be applied to NAT translation.

## **4.6 NAT and PAT Configuration and Verification**

**Configure PAT (overload) on R1 using the WAN interface**

The router R1 uses the previously defined access list (ACL 1) to identify internal IP addresses from the LAN 192.168.10.0/24. These addresses are translated to the public IP of the WAN interface GigabitEthernet0/1 toward the ISP. Port Address Translation (PAT) allows multiple internal hosts to share a single external IP address using different port numbers.

**Configuration summary (R1)**

- ACL 1 defines the internal network to be translated.
    
- GigabitEthernet0/1 acts as the WAN-facing interface used for NAT overload.
    
#### **Command in CLI:**

```
enable
configure terminal
ip nat inside source list 1 interface Gi0/1 overload
end
```
![](images/Pasted%20image%2020251112200227.png)

#### **Testing**

```
PC1> ping 100.64.1.2
```
![](images/Pasted%20image%2020251112200332.png)

A ping from PC1 to the ISP interface (100.64.1.2) generates outbound traffic from the inside network, triggering the NAT translation on R1.

#### **Verification**

```
show ip nat translations
```
![](images/Pasted%20image%2020251112200321.png)

#### **Results**

Displays current NAT translations. For PAT, all internal hosts share the same external IP (100.64.1.1) but use different port numbers.

```
show ip nat statistics
```
![](images/Pasted%20image%2020251112200500.png)
#### **Results**

Displays the number of active translations and packet counts, confirming that NAT and PAT are working correctly.

## 4.7 **Conclusion**

The NAT and PAT configuration on router R1 allows devices in Branch-A to access external networks using a single public IP address. A test ping from PC1 to the ISP confirms that dynamic translations are created correctly, and verification commands show active mappings and packet statistics. The GRE tunnel and Branch-B router remain stable, ensuring uninterrupted communication between branches. This setup demonstrates the correct use of PAT for efficient IP address utilization and reliable network connectivity.


---

