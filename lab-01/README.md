# **Lab 01: Quick Overview **

## **🎯 Core Purpose**
Configure a basic 2-PC network through a switch with device security.

## **🔄 Main Flow**
1. **Switch Setup** → Console access → Hostname → Passwords → SSH
2. **PC Setup** → IP addresses (192.168.1.10 & .20)
3. **Test** → Ping between PCs
4. **Save** → Copy running to startup

## **⚠️ Critical Checks**
- ✓ Ping PC1→PC2 works
- ✓ Switch hostname changed to SW1
- ✓ Passwords set (enable secret + line)
- ✓ Config saved

## **🔧 Key Commands (Remember These)**
```cisco
enable
configure terminal
hostname SW1
enable secret cisco123
copy running-config startup-config
show running-config
```

## **Author & Ownership**

**Maintained by:** **[Sai Aik Kwan](https://www.linkedin.com/in/sai-aik-kwan-251b62396)** | **[Rick's Home Lab](https://www.github.com/rick-homelab)**
> **Start with 'zero' to be the 'hero'.**
