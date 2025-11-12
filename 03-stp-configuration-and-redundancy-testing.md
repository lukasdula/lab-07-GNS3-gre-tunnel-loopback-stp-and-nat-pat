

# **3 - STP Configuration and Redundancy Testing**

## **3.1 Introduction**

This chapter introduces the Spanning Tree Protocol (STP) implementation between the two access switches in Branch A. The network topology includes two switches, SW1 and SW2, connected by redundant links to ensure network reliability and fault tolerance. STP prevents potential switching loops by dynamically blocking one of the redundant paths while keeping the backup link ready for automatic recovery.

In this lab, SW1 acts as the root bridge for VLAN communication within the branch, and SW2 operates as a secondary bridge. The configuration demonstrates how STP maintains a stable Layer 2 topology while allowing redundancy between the two switches.



![](images/Pasted%20image%2020251112192424.png)


## **3.2 Topology**

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


## 3.3 Steps


1. Enable STP on both switches to prevent switching loops.
    
2. Configure SW1 with the lowest bridge priority to make it the root bridge. Verify on both switches that SW1 is the active root and that SW2 recognizes it through BPDU messages.
    
3. Test link redundancy by disconnecting one cable and monitoring reconvergence.
    
4. Record spanning-tree outputs for documentation and verification.

## **3.4 Enable STP on both switches****


Verify that Spanning Tree Protocol (STP) is active on both access switches (SW1 and SW2) to prevent potential switching loops before redundancy testing.

### **Switch (SW1)**

```plaintext
enable
show spanning-tree summary
```
![](images/Pasted%20image%2020251112182424.png)

#### **Results**

Output confirms PVST mode is active. VLAN0001 shows 1 port in the _Blocking_ state and 3 ports in the _Forwarding_ state, verifying that STP is functioning correctly and preventing loops.


### **Switch (SW2)**

```plaintext
enable
show spanning-tree summary
```
![](images/Pasted%20image%2020251112183403.png)

#### **Results**

Output confirms PVST mode is active. All four ports of VLAN0001 are in _Forwarding_ state, indicating that STP synchronization with the root bridge (SW1) is stable and operational.


> **Notes.:** STP (Spanning Tree Protocol) is enabled by default on Cisco switches operating in PVST mode.  
> This verification ensures that both switches are actively participating in the Spanning Tree process before root bridge configuration.  
> No manual mode change (`spanning-tree mode pvst`) is required since PVST is the default operational mode on Cisco IOS.


## **3.5 Set SW1 as the root bridge**


Set SW1 as the root bridge by assigning it the lowest bridge priority for VLAN0001, ensuring it becomes the primary root in the Spanning Tree topology.


### **Switch (SW1)**

```plaintext
enable
configure terminal
spanning-tree vlan 1 priority 4096
end
write memory
```
![](images/Pasted%20image%2020251112184419.png)

The output confirms SW1 is the **Root Bridge** for VLAN0001.  
Root ID and Bridge ID are identical, proving this switch is the root.  
All ports (Gi0/0–Gi0/3) are in **Forwarding** state, which is expected behavior for a root bridge.

![](images/Pasted%20image%2020251112185117.png)

### **Switch (SW2)**

* ***Verification**:

```plaintext
show spanning-tree vlan 1
```
![](images/Pasted%20image%2020251112184513.png)

#### **Results**

SW1 is elected as the Root Bridge for VLAN0001. Bridge ID priority now shows 4097, confirming the manual configuration has overridden the default value (32768). SW2 recognizes SW1 as the root through BPDU messages. One redundant port (Gi0/3) is in the _Blocking_ state, confirming that STP loop prevention is active on SW2.

>**Notes.:** The displayed bridge priority of 4097 is not an error. Cisco adds the VLAN ID (1) to the configured priority (4096) because of the Extended System ID feature. The actual bridge priority remains 4096, but it appears as 4097 in STP outputs when VLAN 1 is included.




## **3.6 Test link redundancy and failover**

Test the Spanning Tree Protocol (STP) failover mechanism by simulating a link failure and verifying that the previously blocked port on SW2 transitions to the Forwarding state.

### **Switch (SW2)**

```plaintext
enable
configure terminal
interface gi0/2
shutdown
```
![](images/Pasted%20image%2020251112190701.png)

**Verification**

```plaintext
show spanning-tree vlan 1
```
![](images/Pasted%20image%2020251112190843.png)

#### **Results**

After shutting down interface Gi0/2, the previously blocked port Gi0/3 automatically transitions to the Forwarding state as it becomes the new Root Port. STP recalculates the topology and restores full connectivity between SW1 and SW2 through the backup link. The network remains stable with no broadcast loops or interruptions in communication, confirming that STP failover is functioning correctly.


> **Notes.:**This test demonstrates STP redundancy and failover capability. When the active link (Gi0/2) fails, the alternate link (Gi0/3) becomes active to maintain the network topology.  
> Recovery time depends on STP timers (listening, learning, forwarding) and typically takes 30–50 seconds in standard STP mode.


## **3.7 Conclusion**

The implementation of Spanning Tree Protocol (STP) between SW1 and SW2 successfully demonstrated loop prevention and link redundancy in a Layer 2 environment. SW1 was configured as the root bridge for VLAN0001, while SW2 acted as the secondary bridge. During the test, STP dynamically blocked one redundant link to prevent switching loops and automatically reactivated the backup link upon simulated failure.

**The results confirmed that:**

- STP correctly elected SW1 as the root bridge.
    
- Redundant links were efficiently managed by blocking one port on SW2.
    
- Failover and reconvergence occurred automatically when the active link was shut down.
    

This chapter validated the stability and resilience of STP under normal and failure conditions, providing a reliable Layer 2 topology that ensures continuous network connectivity and redundancy.


---

Next part: 