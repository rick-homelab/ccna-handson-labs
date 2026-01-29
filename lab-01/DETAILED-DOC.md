# **Lab 01: Basic Device Configuration & Connectivity**

**Status:** ✅ Complete | **Platform:** Cisco Packet Tracer  
**Last Updated:** 1/29/2026

---

## **Table of Contents**
- [🎯 What You're Building Today](#-what-youre-building-today)
- [📖 How This Lab Works - Simple Analogy](#-how-this-lab-works---simple-analogy)
- [🔄 The Big Picture Flow](#-the-big-picture-flow)
- [🔧 Step-by-Step Configuration Explained](#-step-by-step-configuration-explained)
  - [Part 1: Setting Up the Switch - Like Configuring a New Smartphone](#part-1-setting-up-the-switch---like-configuring-a-new-smartphone)
  - [Part 2: Configuring the Computers](#part-2-configuring-the-computers)
  - [Part 3: Testing Connectivity - The "Knock Test"](#part-3-testing-connectivity---the-knock-test)
  - [Part 4: Saving Your Work - Don't Lose Your Configuration!](#part-4-saving-your-work---dont-lose-your-configuration)
- [🔍 Common Problems & Simple Fixes](#-common-problems--simple-fixes)
- [💡 Real-World Application](#-real-world-application)
- [🎓 What You've Actually Learned](#-what-youve-actually-learned)
- [🚀 Ready for the Real World?](#-ready-for-the-real-world)
- [📝 Quick Reference Cheat Sheet](#-quick-reference-cheat-sheet)
- [Author & Ownership](#-author--ownership)
---

## **🎯 What we're Building Today**
We're creating a simple network where two computers can talk to each other through a switch. This is like learning to connect two phones so they can call each other - the foundation of all networking!

## **📖 How This Lab Works - Simple Analogy**

Think of this setup like setting up communication between two rooms in a small office:

| Network Component | Real-World Equivalent | What It Does |
|------------------|----------------------|--------------|
| **PC1 & PC2** | Two office computers | Need to share files and messages |
| **Switch** | Smart office receptionist | Knows which computer is connected to which port and delivers messages |
| **IP Addresses** | Room numbers (192.168.1.10 = Room 10) | Identifies where to send information |
| **Ping Command** | Knocking on a door | Checking if someone is there and can respond |
| **Console Cable** | Master key | Direct access to configure the switch |

## **🔄 The Big Picture Flow**

Here's what happens when PC1 tries to talk to PC2:

```mermaid
graph LR
    PC1[PC1: "Hey PC2,<br/>are you there?"] --> SW1[Switch learns:<br/>"PC1 is on port 1"] --> PC2[PC2: "Yes I'm here!"] --> SW1[Switch learns:<br/>"PC2 is on port 2"] --> PC1
    
    style SW1 fill:#e1f5fe
    style PC1 fill:#f3e5f5
    style PC2 fill:#f3e5f5
```

**What the switch is learning during this conversation:**
1. PC1 sends message → Switch remembers: "PC1 is connected to FastEthernet0/1"
2. Switch floods it out all ports, except the one it was received → PC2 got the message
3. PC2 replies → Switch remembers: "PC2 is connected to FastEthernet0/2"
4. Future messages between them go directly to the right ports

## **🔧 Step-by-Step Configuration Explained**

### **Part 1: Setting Up the Switch - Like Configuring a New Smartphone**

**Step 1-3: Getting Access to the Switch**
```cisco
Switch> enable           ← "Let me use admin features"
Switch#                  ← You're now in admin mode!
Switch# configure terminal  ← "Let me change settings"
Switch(config)#           ← You're now in settings mode
```

**Visual Guide to Modes:**
```
User EXEC Mode       → "Guest Mode" (look only, can't change)
    ↓ enable
Privileged EXEC Mode → "Admin Mode" (look & test)
    ↓ configure terminal
Global Config Mode   → "Settings Mode" (make changes)
```

**Step 4: Naming Your Switch**
```cisco
Switch(config)# hostname SW1
SW1(config)#
```
- Changes the prompt from `Switch#` to `SW1#`
- Important when you have multiple switches
- **Tip:** Use names that tell you location (like `OFFICE-SW1` or `FLOOR2-SW`)

**Step 5: Setting Up a Welcome Message**
```cisco
SW1(config)# banner motd # Authorized Access Only! #
```
- Shows BEFORE anyone logs in
- Legal warning for unauthorized users
- The `#` signs are just markers - they won't show in the actual message
- You can use any character not in your message as markers

**Step 6: Setting the Admin Password**
```cisco
SW1(config)# enable secret cisco123
```
- **Encrypted password** (can't read it from the config file)
- Better than `enable password` (which shows in clear text)
- **Remember:** Passwords are case-sensitive: `Cisco123` ≠ `cisco123`

**Step 7: Securing the Console Port**
```cisco
SW1(config)# username NAME secret PASSWORD      ← "Change 'NAME' and 'PASSWORD' to what you desire"
SW1(config)# line console 0     ← "Let me configure the physical port"
SW1(config-line)# login local          ← "Need to login with the username and secret you had configured"
SW1(config-line)# logging synchronous  ← "Don't interrupt my typing!(line break)"
```

> **The `logging synchronous` command is important because without it, system messages can appear in the middle of what you're typing!**
> **Note:** This lab focuses on console access only. **SSH configuration** will be covered in later labs as we build more advanced skills.
### **Part 2: Configuring the Computers**

**PC1 Settings:**
- **IP Address:** 192.168.1.10 (Think: "Apartment 10")
- **Subnet Mask:** 255.255.255.0 (Think: "Same apartment building")
- **Default Gateway:** 192.168.1.1 (Think: "Main exit door to outside")

**Understanding IP Addresses and Subnets:**
```
192.168.1.10  = Device address (your specific apartment)
255.255.255.0 = Network mask (defines which building you're in)
/24           = Shorthand for 255.255.255.0
               (24 ones in binary: 11111111.11111111.11111111.00000000)
```

**Why Do We Need a Default Gateway?**
- **Same network communication:** PC1→PC2 = No gateway needed
- **Different network communication:** PC1→Google.com = Needs gateway
- Think of it like: Within your building, you can walk to neighbors. To go downtown, you need to use the main exit.

**PC2 Settings:**
- **IP Address:** 192.168.1.20
- **Subnet Mask:** 255.255.255.0
- **Default Gateway:** 192.168.1.1

**Important:** Both computers **MUST** have the same subnet mask and be in the same network range to communicate directly!

### **Part 3: Testing Connectivity - The "Knock Test"**

**Successful Ping - What You Want to See:**
```
PC1> ping 192.168.1.20
Pinging 192.168.1.20 with 32 bytes of data:
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128  ← "PC2 answered!"
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128
Reply from 192.168.1.20: bytes=32 time<1ms TTL=128
Success rate is 100 percent (5/5)  ← Perfect communication!
```

**Failed Ping - What Could Be Wrong:**
```
Request timed out.           ← PC2 didn't answer (check cables/power)
Destination host unreachable ← PC1 doesn't know how to reach PC2 (check IP settings)
```

### **Part 4: Saving Your Work - Don't Lose Your Configuration!**

```cisco
SW1# copy running-config startup-config
```
- **Running-config** = What's in memory (RAM) = Temporary (like working on a document without saving)
- **Startup-config** = What's saved (NVRAM) = Permanent (like saving the document to your hard drive)
- **What happens if you don't save?** = Configuration is lost when switch reboots or loses power!

## **🔍 Common Problems & Simple Fixes**

| Problem | What's Wrong | How to Fix |
|---------|--------------|------------|
| Messages break your typing | Missing `logging synchronous` | Add it to console line configuration |
| Can't get into enable mode | Wrong enable password | Make sure you're using the right password |
| Ping fails but IPs look right | Different subnet masks | Check both PCs use 255.255.255.0 |
| Configuration gone after reboot | Didn't save | Use `copy run start` every time you make changes |
| Can't type anything on console | Cable not connected properly | Check console cable connections |

## **💡 Real-World Application**

**Where You'd Use This Knowledge:**
- **Home Network:** Connecting your computer to a network printer
- **Small Office:** Two computers sharing files
- **Retail Store:** Cash register talking to receipt printer
- **Restaurant:** Kitchen display system getting orders from the front
- **Classroom:** Students sharing files with the teacher

**Why Security Matters:**
- **Without passwords:** Anyone can change your network settings
- **Without saving:** Power outage = start over from scratch
- **Proper configuration:** Ensures reliable communication

## **🎓 What You've Actually Learned**

**Technical Skills:**
1. How to access and configure a network switch
2. How to assign IP addresses to devices
3. How to test basic network connectivity
4. How to secure network devices with passwords
5. How to save configurations

**Problem-Solving Skills:**
1. Methodical approach to configuration
2. Systematic testing methodology
3. Understanding of cause and effect in networks
4. Documentation and saving work

**Big Picture Understanding:**
1. Networks are built in layers
2. Everything connects through switches
3. Security starts with the basics
4. Testing validates your work

## **🚀 Ready for the Real World?**

**You can now:**
- Set up a basic home or small office network
- Troubleshoot "no connection" problems between devices
- Securely configure network equipment
- Document network configurations

**Next steps if you want to practice more:**
1. Add a third PC to the network (use 192.168.1.30)
2. Configure all three PCs to ping each other
3. Add a simple message banner to the switch
4. Practice the configuration from memory

---

## **📝 Quick Reference Cheat Sheet**

```cisco
# BASIC COMMANDS EVERYONE SHOULD KNOW
enable                        # Enter admin mode
configure terminal           # Enter configuration mode
hostname NAME                # Change device name
enable secret PASSWORD       # Set encrypted admin password
copy running-config startup-config  # Save configuration (ALWAYS DO THIS!)

# SECURITY COMMANDS
username NAME secret PASSWORD # Set username and password for console line access
banner motd # MESSAGE #     # Set login warning message
line console 0              # Configure physical port
  login local               # Require username and password to access the device
  logging synchronous       # Stop messages from interrupting typing

# VERIFICATION COMMANDS
show running-config         # View current settings
show ip interface brief     # Check which ports are working
ping IP_ADDRESS             # Test if you can reach another device

# TROUBLESHOOTING
show version                # Check device information
show interfaces status      # See if ports are connected
```

---

## **Author & Ownership**

**Author:** **[Sai Aik Kwan](#https://www.linkedin.com/in/sai-aik-kwan-251b62396/)**  
**Maintained by:** **[Sai Aik Kwan](#https://www.linkedin.com/in/sai-aik-kwan-251b62396/)**
**Working Lab File:** [Download Lab-01](./lab-01.pkt)

*"Every expert was once a beginner. You've just taken your first step into networking - congratulations!"*

---
