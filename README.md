# DHCP-relay-cisco-packet-tracer
DHCP Relay Agent configuration for multiple VLANs
# DHCP Relay Agent Configuration - Cisco Packet Tracer

## 📋 Project Overview
This project demonstrates **DHCP Relay Agent (ip helper-address)** configuration using Cisco Packet Tracer. A single DHCP server on **VLAN 200** dynamically assigns IP addresses to clients on **VLAN 100** across different network segments — a fundamental concept in enterprise network administration.

## 🌐 Network Topology

![Network Topology](screenshots/dhcp-relay-topology.png)

```
                        Router1 (2811)
                       /              \
                      /                \
               Switch0                Switch1 ──── Server0 (DHCP)
              (2960-24TT)            (2960-24TT)
              /        \             /        \
           PC0          PC1       PC2          PC3
         
        ╔══════════╗            ╔══════════╗
        ║  VLAN 100 ║            ║  VLAN 200 ║
        ║ (Clients) ║            ║  (DHCP)   ║
        ╚══════════╝            ╚══════════╝
```

### Network Architecture:
- **Router1** (Cisco 2811) - Central router acting as DHCP Relay Agent
- **Switch0** (Cisco 2960-24TT) - VLAN 100 switch (client network)
- **Switch1** (Cisco 2960-24TT) - VLAN 200 switch (server network)
- **Server0** - DHCP Server providing IPs for both VLANs
- **PC0, PC1** - VLAN 100 clients (receive IPs from DHCP)
- **PC2, PC3** - VLAN 200 clients (receive IPs from DHCP)

---

## 🎯 Objectives
- Configure DHCP server to serve multiple VLANs from a single server
- Implement DHCP Relay Agent using `ip helper-address`
- Set up inter-VLAN routing using Router-on-a-Stick
- Enable automatic IP assignment across different network segments
- Understand how DHCP requests are forwarded across VLANs

---

## 🔧 Configuration

### Step 1: Create VLANs on Switch0
```cisco
Switch0>enable
Switch0#configure terminal

! Create VLAN 100
Switch0(config)#vlan 100
Switch0(config-vlan)#name VLAN100-Clients
Switch0(config-vlan)#exit

! Assign ports to VLAN 100
Switch0(config)#interface FastEthernet0/1
Switch0(config-if)#switchport mode access
Switch0(config-if)#switchport access vlan 100
Switch0(config-if)#exit

Switch0(config)#interface FastEthernet0/2
Switch0(config-if)#switchport mode access
Switch0(config-if)#switchport access vlan 100
Switch0(config-if)#exit

! Configure trunk port to router
Switch0(config)#interface GigabitEthernet0/1
Switch0(config-if)#switchport mode trunk
Switch0(config-if)#exit
```

### Step 2: Create VLANs on Switch1
```cisco
Switch1>enable
Switch1#configure terminal

! Create VLAN 200
Switch1(config)#vlan 200
Switch1(config-vlan)#name VLAN200-Server
Switch1(config-vlan)#exit

! Assign PC ports to VLAN 200
Switch1(config)#interface FastEthernet0/1
Switch1(config-if)#switchport mode access
Switch1(config-if)#switchport access vlan 200
Switch1(config-if)#exit

Switch1(config)#interface FastEthernet0/2
Switch1(config-if)#switchport mode access
Switch1(config-if)#switchport access vlan 200
Switch1(config-if)#exit

! Assign server port to VLAN 200
Switch1(config)#interface FastEthernet0/3
Switch1(config-if)#switchport mode access
Switch1(config-if)#switchport access vlan 200
Switch1(config-if)#exit

! Configure trunk port to router
Switch1(config)#interface GigabitEthernet0/1
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#exit
```

### Step 3: Configure Router1 (Router-on-a-Stick)
```cisco
Router1>enable
Router1#configure terminal

! Enable main interface
Router1(config)#interface GigabitEthernet0/0
Router1(config-if)#no shutdown
Router1(config-if)#exit

! Subinterface for VLAN 100
Router1(config)#interface GigabitEthernet0/0.100
Router1(config-subif)#encapsulation dot1Q 100
Router1(config-subif)#ip address 192.168.100.1 255.255.255.0
Router1(config-subif)#ip helper-address 192.168.200.10
Router1(config-subif)#exit

! Subinterface for VLAN 200
Router1(config)#interface GigabitEthernet0/0.200
Router1(config-subif)#encapsulation dot1Q 200
Router1(config-subif)#ip address 192.168.200.1 255.255.255.0
Router1(config-subif)#exit
```

> 💡 **Key Command**: `ip helper-address 192.168.200.10` on the VLAN 100 subinterface tells the router to forward all DHCP requests from VLAN 100 to the DHCP server at 192.168.200.10

### Step 4: Configure DHCP Server (Router CLI)
```cisco
! Exclude gateway IPs from DHCP pools
Router1(config)#ip dhcp excluded-address 192.168.100.1
Router1(config)#ip dhcp excluded-address 192.168.200.1

! DHCP Pool for VLAN 100
Router1(config)#ip dhcp pool VLAN100-Pool
Router1(dhcp-config)#network 192.168.100.0 255.255.255.0
Router1(dhcp-config)#default-router 192.168.100.1
Router1(dhcp-config)#dns-server 8.8.8.8
Router1(dhcp-config)#exit

! DHCP Pool for VLAN 200
Router1(config)#ip dhcp pool VLAN200-Pool
Router1(dhcp-config)#network 192.168.200.0 255.255.255.0
Router1(dhcp-config)#default-router 192.168.200.1
Router1(dhcp-config)#dns-server 8.8.8.8
Router1(dhcp-config)#exit
```

### Step 5: Configure Server0 (DHCP Server Device)
On the Server0 device in Packet Tracer:
1. Click **Server0 → Services → DHCP**
2. Turn service **ON**
3. Add Pool for VLAN 100:
```
Pool Name:        VLAN100-Pool
Default Gateway:  192.168.100.1
DNS Server:       8.8.8.8
Start IP:         192.168.100.10
Subnet Mask:      255.255.255.0
Max Users:        50
```
4. Click **Add**
5. Add Pool for VLAN 200:
```
Pool Name:        VLAN200-Pool
Default Gateway:  192.168.200.1
DNS Server:       8.8.8.8
Start IP:         192.168.200.10
Subnet Mask:      255.255.255.0
Max Users:        50
```
6. Click **Add** then **Save**

---

## 📊 IP Addressing Scheme

### Network Summary
| Network | VLAN | Subnet | Gateway | DHCP Range |
|---------|------|--------|---------|------------|
| Client Network | VLAN 100 | 192.168.100.0/24 | 192.168.100.1 | 192.168.100.10 - .254 |
| Server Network | VLAN 200 | 192.168.200.0/24 | 192.168.200.1 | 192.168.200.10 - .254 |

### Device IP Assignments
| Device | VLAN | IP Address | Assignment |
|--------|------|------------|------------|
| Router1 Gi0/0.100 | 100 | 192.168.100.1 | Static |
| Router1 Gi0/0.200 | 200 | 192.168.200.1 | Static |
| Server0 | 200 | 192.168.200.10 | Static |
| PC0 | 100 | 192.168.100.10 | DHCP ✅ |
| PC1 | 100 | 192.168.100.11 | DHCP ✅ |
| PC2 | 200 | 192.168.200.11 | DHCP ✅ |
| PC3 | 200 | 192.168.200.12 | DHCP ✅ |

---

## ✅ Verification & Testing

### Check DHCP Bindings
```cisco
Router1#show ip dhcp binding
```
Expected output:
```
IP Address      Client-ID                Lease Expiration
192.168.100.10  0100.50XX.XXXX.XX       --
192.168.100.11  0100.50YY.YYYY.YY       --
192.168.200.11  0100.50ZZ.ZZZZ.ZZ       --
192.168.200.12  0100.50AA.AAAA.AA       --
```

### Check DHCP Pool Status
```cisco
Router1#show ip dhcp pool
```

### Check Interface Status
```cisco
Router1#show ip interface brief
```

### Check VLAN Status on Switch
```cisco
Switch0#show vlan brief
Switch1#show vlan brief
```

### Check Trunk Ports
```cisco
Switch0#show interfaces trunk
Switch1#show interfaces trunk
```

### Test Connectivity
From PC0 (VLAN 100):
```
C:\>ipconfig          (verify DHCP assigned IP)
C:\>ping 192.168.200.10   (ping DHCP server)
C:\>ping 192.168.100.11   (ping PC1 same VLAN)
C:\>ping 192.168.200.11   (ping PC2 different VLAN)
```

---

## 🔄 How DHCP Relay Works

```
1. PC0 sends DHCP Discover (broadcast) on VLAN 100
         ↓
2. Router1 Gi0/0.100 receives the broadcast
         ↓
3. ip helper-address converts broadcast to unicast
   and forwards to DHCP Server (192.168.200.10)
         ↓
4. DHCP Server checks which pool matches
   (192.168.100.x pool for VLAN 100 requests)
         ↓
5. DHCP Server sends IP offer back to Router1
         ↓
6. Router1 forwards offer back to PC0
         ↓
7. PC0 receives IP: 192.168.100.10 ✅
```

---

## 🛠️ Equipment Used
- **Router**: Cisco 2811 (x1)
- **Switches**: Cisco 2960-24TT (x2)
- **Server**: Server-PT (x1) - DHCP Server
- **End Devices**: PC-PT (x4)
- **Cables**: Copper Straight-Through

---

## 📚 Key Concepts Demonstrated

### 1. DHCP Relay Agent
- Forwards DHCP broadcasts across VLANs using `ip helper-address`
- Eliminates need for a DHCP server on every subnet
- Converts DHCP broadcasts to unicast packets

### 2. Router-on-a-Stick
- Single physical router interface serving multiple VLANs
- Uses subinterfaces with 802.1Q encapsulation (dot1Q)
- Cost-effective inter-VLAN routing solution

### 3. VLAN Segmentation
- VLAN 100: Client network (PC0, PC1)
- VLAN 200: Server network (PC2, PC3, Server0)
- Trunk ports carry multiple VLANs between switch and router

### 4. Centralized DHCP Management
- Single DHCP server serving multiple VLANs
- Separate IP pools per VLAN
- Automatic IP assignment for all clients

---

## 🔍 Troubleshooting Guide

| Problem | Possible Cause | Solution |
|---------|---------------|----------|
| PC not getting IP | `ip helper-address` missing | Add `ip helper-address` on VLAN 100 subinterface |
| Wrong IP assigned | DHCP pool mismatch | Check pool gateway matches subinterface IP |
| Can't ping across VLANs | Trunk not configured | Verify trunk on switch uplink port |
| DHCP server unreachable | Routing issue | Ping server from router first |
| IP conflict | Excluded address missing | Add `ip dhcp excluded-address` for static IPs |

---

## 🚀 How to Run This Project
1. Download and install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer)
2. Clone this repository
3. Open `dhcp-relay.pkt` in Packet Tracer
4. Set all PCs to **DHCP** mode (Desktop → IP Configuration → DHCP)
5. Verify PCs receive correct IPs using `ipconfig`
6. Test cross-VLAN connectivity using `ping`

---

## 📁 Repository Structure
```
dhcp-relay-cisco/
├── README.md
├── dhcp-relay.pkt
├── screenshots/
│   ├── dhcp-relay-topology.png
│   ├── dhcp-binding.png
│   └── ping-test.png
└── configs/
    ├── router1-config.txt
    ├── switch0-config.txt
    └── switch1-config.txt
```

---

## 💡 Learning Outcomes
- Understanding DHCP Relay Agent operation
- Configuring `ip helper-address` on Cisco routers
- Implementing Router-on-a-Stick for inter-VLAN routing
- Managing centralized DHCP for multiple VLANs
- Troubleshooting DHCP and VLAN issues

---

## 📜 License
This project is for educational purposes.

---
**Note**: This project demonstrates enterprise-grade DHCP relay configuration — a critical skill for network administrators managing large multi-VLAN environments.
