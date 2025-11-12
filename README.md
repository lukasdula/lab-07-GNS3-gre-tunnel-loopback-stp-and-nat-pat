# **GRE Tunnel, STP, Loopback, and NAT/PAT**

## **Introduction and Objectives**

This lab connects two separate branch networks through an ISP, using WAN addressing, static routing, GRE tunneling, loopback interfaces, STP redundancy, and NAT/PAT translation. The goal is to create a functional inter-branch topology that maintains communication between LANs and external networks. The configuration gradually builds from basic IP addressing and routing to tunnel setup, redundancy testing, and address translation on R1.


![](images/Pasted%20image%2020251112203756.png)



The objectives are:

- Configure WAN and LAN addressing between routers, switches, and hosts
    
- Establish static routes and GRE tunnel between Branch-A and Branch-B
    
- Implement STP redundancy between SW1 and SW2 in Branch-A
    
- Configure NAT and PAT on R1 to allow access to external networks
    
- Verify connectivity and stability across all network layers
    



## **Lab Structure**

1. [WAN and Addressing Setup](01-wan-and-addressing-setup.md)
    
2. [GRE Tunnel and Loopback Configuration](02-gre-tunnel-and-loopback-configuration.md)
    
3. [STP Configuration and Redundancy Testing](03-stp-configuration-and-redundancy-testing.md)
    
4. [NAT and PAT Configuration and Verification](04-nat-and-pat-configuration-and-verification.md)
    



## **Key Features**

- WAN and LAN addressing with static routing between branches
    
- GRE tunnel providing logical Layer 3 connection through ISP
    
- Loopback interfaces used for tunnel source and testing
    
- STP ensuring link redundancy and loop prevention in Branch-A
    
- NAT and PAT enabling outbound traffic translation on R1
    
- Verification through ping tests and translation monitoring
    



## **Author’s Note**

This lab was my first experience with connecting two separate network areas using a GRE tunnel. Configuring the tunnel between R1 and R2 helped me understand how routing can extend beyond a single local network. Working with STP was also very interesting; seeing how it automatically blocks and reactivates redundant links showed how reliable Layer 2 protection can be.

Through this project, I learned how different technologies like loopbacks, tunnels, STP, and NAT can work together to create a functioning network. Building the structure that connected two remote areas and testing all these functions was both useful and rewarding.


---

© 2025 – Lukas Dula | Home Network Lab & Portfolio
