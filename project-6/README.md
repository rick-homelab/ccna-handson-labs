# **Project 6: Basic Static Routing**

**Time Estimate:** 45 minutes | **Difficulty:** Beginner-Intermediate | **Status:** Tested ✓ | **Last Updated:** 2025-10-5

## **Table of Contents**
- [Objective](#objective)
- [Quick Start](#quick-start)
- [Topology & Design](#topology--design)
- [Configuration](#configuration)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)
- [Protocol Deep Dive](#protocol-deep-dive)
- [Skills Demonstrated](#skills-demonstrated)
- [Real-World Applications](#real-world-applications)
- [Learning Outcomes](#learning-outcomes)

## **Objective**
> Connect three separate networks using static routing to understand manual route configuration, multi-hop packet forwarding, and routing table management in multi-router environments.

**Key Goals:**
- [ ] Configure router interfaces for multiple networks
- [ ] Implement static routes for remote network reachability
- [ ] Establish end-to-end communication across routers
- [ ] Understand routing table population and packet forwarding

## **Quick Start**
### **Prerequisites**
- Completion of Project 3 (Basic router configuration)
- Understanding of IP addressing and subnetting
- Packet Tracer with multiple routers

### **Implementation Checklist**
- [ ] Configure R1 interfaces and static route (15 minutes)
- [ ] Configure R2 interfaces and static route (15 minutes)
- [ ] Set up PC IP addresses and gateways (5 minutes)
- [ ] Test multi-hop connectivity (10 minutes)

### **Time Breakdown**
| Phase | Time | Focus |
|-------|------|-------|
| R1 Config | 15 min | Interfaces + static route |
| R2 Config | 15 min | Interfaces + static route |
| PC Setup | 5 min | IP and gateway configuration |
| Verification | 10 min | Cross-network testing |

## **Topology & Design**

### **Network Architecture**
```mermaid
graph TB
    subgraph "Network A: 192.168.1.0/24<br/>HQ LAN"
        PC1[PC1<br/>192.168.1.10/24]
        PC2[PC2<br/>192.168.1.11/24]
        SW1[Switch1<br/>2960-24TT]
    end
    
    subgraph "Network B: 10.0.0.0/30<br/>WAN Link"
        R1[R1: 1941 Router<br/>G0/0: 192.168.1.1/24<br/>G0/1: 10.0.0.1/30]
        R2[R2: 1941 Router<br/>G0/0: 10.0.0.2/30<br/>G0/1: 192.168.2.1/24]
    end
    
    subgraph "Network C: 192.168.2.0/24<br/>Branch LAN"
        PC3[PC3<br/>192.168.2.10/24]
        PC4[PC4<br/>192.168.2.11/24]
        SW2[Switch2<br/>2960-24TT]
    end
    
    PC1 -->|straight-through| SW1
    PC2 -->|straight-through| SW1
    SW1 -->|straight-through| R1
    R1 -->|crossover| R2
    R2 -->|straight-through| SW2
    SW2 -->|straight-through| PC3
    SW2 -->|straight-through| PC4
```

### **Packet Flow Visualization**
```mermaid
sequenceDiagram
    participant PC1 as PC1 (Network A)<br/>192.168.1.10
    participant R1 as R1: 1941 Router<br/>G0/0: 192.168.1.1<br/>G0/1: 10.0.0.1
    participant R2 as R2: 1941 Router<br/>G0/0: 10.0.0.2<br/>G0/1: 192.168.2.1
    participant PC3 as PC3 (Network C)<br/>192.168.2.10

    Note over PC1,PC3: PC1 pings PC3 (192.168.2.10)
    
    PC1->>R1: Packet to 192.168.2.10<br/>Via gateway 192.168.1.1
    Note over PC1: • Destination: 192.168.2.10<br/>• Gateway: 192.168.1.1
    
    R1->>R2: Static route: 192.168.2.0/24 via 10.0.0.2
    Note over R1: • Checks routing table<br/>• Static route found<br/>• Forwards to 10.0.0.2
    
    R2->>PC3: Direct delivery to 192.168.2.10
    Note over R2: • Connected network<br/>• Direct delivery<br/>• Decrements TTL
    
    PC3->>R2: Reply to 192.168.1.10
    R2->>R1: Static route: 192.168.1.0/24 via 10.0.0.1
    R1->>PC1: Direct delivery to 192.168.1.10
```

### **Network Design Table**
| Network | Subnet | Purpose | Devices |
|---------|--------|---------|---------|
| Network A | 192.168.1.0/24 | HQ LAN | PC1, PC2, Switch1, R1 G0/0 |
| Network B | 10.0.0.0/30 | WAN Link | R1 G0/1, R2 G0/0 |
| Network C | 192.168.2.0/24 | Branch LAN | PC3, PC4, Switch2, R2 G0/1 |

### **Device Specifications**
| Device | Model | Role | Key Interfaces |
|--------|-------|------|----------------|
| R1 | Cisco 1941 | HQ Router | G0/0: 192.168.1.1/24, G0/1: 10.0.0.1/30 |
| R2 | Cisco 1941 | Branch Router | G0/0: 10.0.0.2/30, G0/1: 192.168.2.1/24 |
| Switch1 | 2960-24TT | HQ Access | Fa0/1-24 for PCs |
| Switch2 | 2960-24TT | Branch Access | Fa0/1-24 for PCs |

### **The WHY**
- **Why /30 for WAN link?** Efficient use of IP space for point-to-point connections
- **Why 1941 routers?** Enterprise-grade with Gigabit Ethernet interfaces
- **Why static routing?** Demonstrates manual route configuration fundamentals
- **Why multiple end devices?** Real-world network simulation

## **Configuration**

### **R1 Configuration (HQ Router)**
```bash
R1> enable
R1# configure terminal

! Configure LAN interface (Network A)
R1(config)# interface gigabitethernet 0/0
R1(config-if)# description HQ-LAN-Network-A
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

! Configure WAN interface (Network B)
R1(config)# interface gigabitethernet 0/1
R1(config-if)# description WAN-Link-to-R2
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit

! Add static route to Network C via R2
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
R1(config)# end
R1# copy running-config startup-config
```

### **R2 Configuration (Branch Router)**
```bash
R2> enable
R2# configure terminal

! Configure WAN interface (Network B)
R2(config)# interface gigabitethernet 0/0
R2(config-if)# description WAN-Link-to-R1
R2(config-if)# ip address 10.0.0.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit

! Configure LAN interface (Network C)
R2(config)# interface gigabitethernet 0/1
R2(config-if)# description Branch-LAN-Network-C
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit

! Add static route to Network A via R1
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
R2(config)# end
R2# copy running-config startup-config
```

### **Switch Configurations**
```bash
! Basic switch configuration (both switches)
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# exit
SW1# copy running-config startup-config
```

### **PC Configurations**
```bash
# Network A - HQ Devices:
PC1: 192.168.1.10/24 | Gateway: 192.168.1.1
PC2: 192.168.1.11/24 | Gateway: 192.168.1.1

# Network C - Branch Devices:
PC3: 192.168.2.10/24 | Gateway: 192.168.2.1
PC4: 192.168.2.11/24 | Gateway: 192.168.2.1
```

### **The WHY**
- **Why /30 mask?** Provides exactly 2 usable IPs for point-to-point link
- **Why interface descriptions?** Professional practice for documentation
- **Why static route syntax?** `ip route [network] [mask] [next-hop]`
- **Why save configuration?** Prevents route loss after router reboot

## **Verification**

### **Expected Results**
```bash
# R1 routing table:
R1# show ip route
Codes: C - connected, S - static, I - IGRP, R - RIP...
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
C    10.0.0.0/30 is directly connected, GigabitEthernet0/1
S    192.168.2.0/24 [1/0] via 10.0.0.2

# Successful end-to-end ping:
C:\> ping 192.168.2.10
Reply from 192.168.2.10: bytes=32 time=4ms TTL=126

# Traceroute shows path:
C:\> tracert 192.168.2.10
Tracing route to 192.168.2.10 over a maximum of 30 hops:
  1   1 ms    1 ms    1 ms    192.168.1.1
  2   2 ms    2 ms    2 ms    10.0.0.2
  3   3 ms    2 ms    2 ms    192.168.2.10
```

### **Verification Steps**
1. **Interface Status:** `show ip interface brief` on both routers
2. **Routing Tables:** `show ip route` to verify static routes
3. **Direct Connectivity:** Ping between router interfaces
4. **End-to-End Test:** Ping from PC1 to PC3 and vice versa
5. **Path Verification:** Traceroute to confirm packet path

### **The WHY**
- **Why check routing tables?** Confirms routers know how to reach remote networks
- **Why traceroute?** Visualizes the actual path packets take
- **Why bidirectional testing?** Ensures routing works in both directions
- **Why TTL analysis?** Shows number of hops (decreases by 1 per router)

## **Troubleshooting**

### **Common Issues & Solutions**
| Symptom | Possible Cause | Solution |
|---------|---------------|----------|
| Ping fails between networks | Missing static route | Verify `show ip route` on both routers |
| One-way communication | Asymmetric routing | Check static routes in both directions |
| Router can't ping next-hop | Interface down | Verify `no shutdown` and physical connections |
| TTL expired in transit | Routing loop | Check for circular static routes |

### **Debug Commands**
```bash
# Router troubleshooting:
show ip route
show ip interface brief
ping 10.0.0.2 (test next-hop reachability)
traceroute 192.168.2.10
show running-config | include ip route

# PC troubleshooting:
ipconfig /all
ping 192.168.1.1 (test gateway)
tracert 192.168.2.10
```

## **Protocol Deep Dive**

### **Routing Decision Process**
```mermaid
graph TB
    A[Router Receives Packet] --> B[Check Destination IP]
    B --> C[Search Routing Table]
    C --> D{Longest Prefix Match}
    D -->|Match Found| E[Forward to Next-Hop]
    D -->|No Match| F[Drop Packet]
    E --> G[Decrement TTL]
    G --> H[Update Checksum]
    H --> I[Forward Packet]
```

### **Static Route Operation**
```mermaid
graph LR
    A[Packet Arrives] --> B[Extract Destination IP]
    B --> C[Search Routing Table]
    C --> D{Static Route Match?}
    D -->|Yes| E[Forward to Next-Hop]
    D -->|No| F[Check Connected Routes]
    F --> G{Connected Match?}
    G -->|Yes| H[Forward Directly]
    G -->|No| I[Drop Packet - No Route]
```

### **Key Concepts**
- **Longest Prefix Match:** Router chooses most specific route available
- **Administrative Distance:** Static routes have AD of 1 (highly trusted)
- **TTL Mechanism:** Prevents infinite loops in routing
- **Recursive Lookup:** Router may need multiple table lookups for final forwarding

## **Skills Demonstrated**
- ✅ **Static Route Configuration** - Manual routing table population
- ✅ **Multi-Router Design** - Network segmentation and connectivity
- ✅ **Routing Table Analysis** - Interpreting and verifying routes
- ✅ **End-to-End Troubleshooting** - Diagnosing multi-hop issues
- ✅ **Network Path Analysis** - Using traceroute for path verification

## **Real-World Applications**

### **Enterprise Network Scenarios**
- **Branch Office Connectivity** - Connecting remote sites to headquarters
- **Network Segmentation** - Controlled routing between departments
- **DMZ Networks** - Specific routing for security zones
- **Backup Links** - Manual failover paths for redundancy

### **Business Value**
- **Predictable Routing** - Complete control over network paths
- **Security** - No routing protocol vulnerabilities
- **Resource Efficiency** - Lower CPU and bandwidth usage
- **Cost Effective** - Suitable for small to medium networks

## **Learning Outcomes**
By completing this lab, you will understand:

### **Technical Knowledge**
- How routers make forwarding decisions using routing tables
- Static vs dynamic routing trade-offs
- Multi-hop packet forwarding principles
- Routing table structure and interpretation

### **Practical Skills**
- Configuring and verifying static routes
- Troubleshooting end-to-end connectivity issues
- Analyzing network paths using traceroute
- Managing routing in multi-router environments

### **Career Foundation**
- **Prepares for:** Dynamic routing protocols (OSPF, EIGRP)
- **Builds toward:** Complex enterprise network designs
- **Essential for:** Network administrator and engineer roles
- **Interview ready:** Fundamental networking concept

---

**Maintained by:** Rick's Home Lab  
*Part of the CCNA Fundamentals Series - Mastering Routing Concepts*

