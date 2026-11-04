# 🌐 VLANs Part 3 — Advanced VLAN Topics

> **Course:** CCNA 200-301 | **Source:** Jeremy's IT Lab
**Tags:** #CCNA #Networking #VLANs #ROAS #Layer3Switching #SVI #8021Q
> 

---

## 📋 Video Overview

This lesson covers advanced VLAN topics for CCNA:

| # | Topic |
| --- | --- |
| 1 | Native VLAN configuration on routers (**ROAS**) |
| 2 | Wireshark captures showing **802.1Q tags** |
| 3 | Layer 3 switching (**Multilayer switching**) for inter-VLAN routing |
| 4 | Introduction to **SVIs** (Switch Virtual Interfaces) |

> 📝 **Note:** DTP and VTP are reserved for a separate video (removed from CCNA exam topics but still useful to understand).
> 

---

## 🏷️ Native VLAN on a Router (ROAS)

### ❓ What is the Native VLAN Benefit?

- Frames in the **native VLAN are untagged** → **more efficient**
- **Smaller frames = more frames per second**

> ⚠️ **Security Best Practice:** Set native VLAN to an **unused VLAN** for security purposes. (Prevents VLAN hopping attacks.)
> 

---

### 🛠️ Two Methods to Configure Native VLAN on a Router

#### 🔹 Method 1: Subinterface with `NATIVE` Keyword

```
interface g0/0.10
 encapsulation dot1q 10 native
```

> Uses the subinterface approach — the `native` keyword tells the router that VLAN 10 is the native VLAN.
> 

#### 🔹 Method 2: Configure IP on Physical Interface

```
interface g0/0
 ip address 192.168.1.1 255.255.255.192
```

> ✅ No subinterface needed
✅ No `encapsulation dot1q` command required
The physical interface naturally handles untagged (native VLAN) traffic.
> 

---

## 🔍 Wireshark Capture Analysis

### 📦 Tagged Frame (VLAN 20 — NOT Native)

| Field | Value | Meaning |
| --- | --- | --- |
| **TPID** | `8100` | Indicates 802.1Q tag |
| **PCP** | `0` | No special priority |
| **DEI** | `0` | Won't be dropped during congestion |
| **VLAN ID** | `20` | The VLAN this frame belongs to |

> 💡 **802.1Q Tag Breakdown:**
> 
> - **TPID** (Tag Protocol Identifier): `0x8100` — identifies the frame as 802.1Q tagged
> - **PCP** (Priority Code Point): 3 bits — QoS priority (0–7)
> - **DEI** (Drop Eligible Indicator): 1 bit — indicates drop eligibility during congestion
> - **VLAN ID**: 12 bits — identifies the VLAN (1–4094)

---

### 📦 Untagged Frame (VLAN 10 — Native VLAN)

- ❌ **No 802.1Q tag present**
- Both router and switch understand untagged frames = **VLAN 10**
- ✅ **More efficient transmission** (smaller frame size)

---

### 📊 Frame Flow Example

> **Scenario:** PC in VLAN 20 pings PC in VLAN 10
> 

```
1. PC (VLAN 20) sends ICMP request
2. SW2 → R1: ICMP request is TAGGED (VLAN 20)
3. R1 → SW2: ICMP request is UNTAGGED (native VLAN 10)
4. Continues untagged to destination
5. Reply follows reverse path
```

> 💡 **Key Insight:** Traffic is tagged when crossing the trunk for non-native VLANs. Native VLAN traffic is always **untagged**.
> 

---

## 🔄 Layer 3 Switching (Multilayer Switching)

### ❓ What is a Multilayer Switch?

| Feature | Description |
| --- | --- |
| **Switching + Routing** | Capable of both Layer 2 switching AND Layer 3 routing |
| **Layer 3 Aware** | Unlike regular Layer 2 switches |
| **IP on Interfaces** | Can assign IP addresses to interfaces |
| **Routed Ports** | Can configure routed ports (`no switchport`) |
| **SVIs** | Can create virtual interfaces for each VLAN |
| **Static Routes** | Supports static routing |
| **Inter-VLAN Routing** | Used for routing between VLANs |

---

### ⚖️ Inter-VLAN Routing Methods Comparison

| Method | ✅ Pros | ❌ Cons |
| --- | --- | --- |
| **Router with Multiple Connections** | Simple | Limited by router interfaces |
| **Router on a Stick (ROAS)** | Efficient interface usage | Network congestion possible |
| **Layer 3 Switch (MLS)** | Best for large networks | More complex configuration |

---

### 📊 Traffic Path Comparison

#### 🔹 Router on a Stick (ROAS)

```
PC (VLAN 20) → SW2 → R1 (tagged VLAN 20) → SW2 (tagged VLAN 10) → SW1 → PC (VLAN 10)
```

#### 🔹 Layer 3 Switch (MLS)

```
PC (VLAN 20) → SW2 (routes internally) → SW1 (tagged VLAN 10) → PC (VLAN 10)
```

> ✅ **Key Advantage:** Traffic **never goes to R1** for inter-VLAN routing! The Layer 3 switch handles routing internally.
> 

---

## ⚙️ Layer 3 Switch Configuration

### 🔹 Step 1: Configure Point-to-Point Link (SW2 ↔ R1)

#### On R1 (Remove ROAS config, set physical interface):

```
no interface g0/0.10
no interface g0/0.20
no interface g0/0.30
default interface g0/0
interface g0/0
 ip address 192.168.1.194 255.255.255.252
```

#### On SW2 (Configure routed port + enable routing):

```
default interface g0/1
ip routing                          ! ← CRITICAL! Enables Layer 3 routing globally
interface g0/1
 no switchport                      ! ← Converts to routed port
 ip address 192.168.1.193 255.255.255.252
```

> ⚠️ **Important:** Don't forget `ip routing` or inter-VLAN routing **won't work!**
> 
> 
> ⚠️ **Important:** `no switchport` converts the interface from a Layer 2 switchport to a Layer 3 routed port, allowing you to configure an IP address.
> 

---

### 🔹 Step 2: Configure Default Route on SW2

```
ip route 0.0.0.0 0.0.0.0 192.168.1.194
```

> Routes all unknown traffic to R1 (the next hop).
> 

---

### 🔹 Step 3: Configure SVIs (Switch Virtual Interfaces)

```
interface vlan10
 ip address 192.168.1.62 255.255.255.192
 no shutdown

interface vlan20
 ip address 192.168.1.126 255.255.255.192
 no shutdown

interface vlan30
 ip address 192.168.1.190 255.255.255.192
 no shutdown
```

> 📝 **Remember:** SVIs are **shutdown by default** — use `no shutdown`!
> 

---

## 📜 SVI Conditions for UP/UP Status

> An SVI requires **ALL** of the following to be **UP/UP**:
> 

| # | Condition | Notes |
| --- | --- | --- |
| 1️⃣ | **VLAN must exist** on the switch | Switch won't auto-create VLAN for SVI |
| 2️⃣ | At least one **access port in the VLAN is up/up**, OR a **trunk port** allowing that VLAN is up/up | Different from SVI shutdown |
| 3️⃣ | **VLAN must not be shutdown** | `no shutdown` on VLAN |
| 4️⃣ | **SVI must not be shutdown** | `no shutdown` on SVI interface |

---

### ⚠️ Important Distinction

| Action | Result |
| --- | --- |
| Assigning an **access port** to a VLAN | ✅ **Auto-creates** the VLAN |
| Creating an **SVI** for a non-existent VLAN | ❌ **Does NOT auto-create** the VLAN |

> 💡 **Key Takeaway:** SVIs need the VLAN to already exist. Access port assignment creates the VLAN automatically, but SVI creation does not.
> 

---

## 🔍 Verification Commands

```
show ip interface brief
```

> Shows all interfaces, IP addresses, and status (up/up or down/down).
> 

```
show interfaces status
```

> Look for **"routed"** in the VLAN column — indicates a routed port.
> 

```
show ip route
```

> Shows connected/local routes for SVIs.
> 

```
show vlan brief
```

> Verify VLANs exist and ports are assigned.
> 

```
show interfaces trunk
```

> Verify trunk configuration and native VLAN settings.
> 

---

## 🔑 Key Commands Summary

| Command | Mode | Purpose |
| --- | --- | --- |
| `encapsulation dot1q [vlan-id] native` | Subinterface Config | Configure native VLAN on subinterface |
| `ip routing` | Global Config | Enable Layer 3 routing on switch |
| `no switchport` | Interface Config | Convert interface to routed port |
| `interface vlan [id]` | Global Config | Create/configure SVI |
| `no shutdown` | Interface Config | Enable SVI (or VLAN) |
| `ip route 0.0.0.0 0.0.0.0 [next-hop]` | Global Config | Configure default route |
| `show ip interface brief` | Privileged EXEC | Verify interface IP and status |
| `show interfaces status` | Privileged EXEC | Verify routed port status |
| `show ip route` | Privileged EXEC | Verify routing table |
| `show interfaces trunk` | Privileged EXEC | Verify trunk and native VLAN |

---

### 🧰 Additional Useful Syntaxes (Bonus for CCNA)

| Command | Mode | Purpose |
| --- | --- | --- |
| `switchport trunk encapsulation dot1q` | Interface Config | Set trunk encapsulation to 802.1Q |
| `switchport mode trunk` | Interface Config | Configure port as trunk |
| `switchport trunk native vlan [id]` | Interface Config | Change native VLAN on trunk |
| `switchport trunk allowed vlan [id]` | Interface Config | Restrict VLANs on trunk |
| `show spanning-tree vlan [id]` | Privileged EXEC | Verify STP per VLAN |
| `show mac address-table` | Privileged EXEC | Verify MAC address table |
| `show ip arp` | Privileged EXEC | Verify ARP table |
| `ping [ip]` | Privileged EXEC | Test connectivity |
| `traceroute [ip]` | Privileged EXEC | Trace path to destination |
| `debug ip packet` | Privileged EXEC | Debug packet forwarding (use with caution) |

---

## 📝 Quiz Questions

### ❓ Question 1

**Which TWO answers are valid options to configure the native VLAN on a router in a ROAS configuration?** *(select two, each answer is a complete solution)*

| Option | Answer |
| --- | --- |
| A) Configure `encapsulation dot1q 10` on subinterface | ❌ |
| B) Configure `encapsulation dot1q 10 native` on subinterface | ✅ |
| C) Configure IP address on physical interface (no subinterface) | ✅ |
| D) Configure `switchport trunk native vlan 10` on router | ❌ |

> **✅ Answer: B and C**
> 
> - **B:** Uses `encapsulation dot1q 10 native` on the subinterface
> - **C:** Configures IP address on the physical interface (no encapsulation command needed)

---

### ❓ Question 2

**You create an SVI for VLAN 225 on SW1, assign an IP address, and enable it with `no shutdown`, but the interface remains down/down. Which TWO options might be causing this?** *(select two)*

| Option | Answer |
| --- | --- |
| A) VLAN 225 doesn't exist on the switch | ✅ |
| B) You didn't issue `switchport mode trunk` on VLAN 225's SVI | ❌ |
| C) You didn't issue `switchport access vlan 225` on VLAN 225's SVI | ❌ |
| D) No interfaces in VLAN 225 are up/up | ✅ |

> **✅ Answer: A and D**
> 
> - **A:** The VLAN must exist on the switch for the SVI to come up
> - **D:** There must be at least one access port or trunk port in that VLAN that is up/up
> 
> ❌ **B and C are incorrect** — you don't configure `switchport` commands on SVIs. SVIs are Layer 3 interfaces.
> 

---

### ❓ Question 3

**Which command is used to configure a switch interface as a routed port?**

| Option | Answer |
| --- | --- |
| A) `no switchport` | ✅ |
| B) `ip address [address] [mask]` | ❌ |
| C) `ip routing` | ❌ |
| D) `switchport mode route` | ❌ |

> **✅ Answer: A) `no switchport`**
> 
> 
> This command converts the interface from a Layer 2 switchport to a Layer 3 routed port, allowing you to configure an IP address.
> 
> ❌ **C (`ip routing`)** enables routing globally on the switch but doesn't set individual interfaces as routed ports.
> 

---

### 🎯 Bonus Question (Boson ExSim)

**You issue the following commands on a Catalyst 2950 switch:**

```
configure terminal
interface FastEthernet 0/7
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 44
```

**Which statement is true regarding VLAN traffic when sent over port FastEthernet0/7?**

| Option | Answer |
| --- | --- |
| A) VLAN 1 traffic will be untagged | ❌ |
| B) VLAN 44 traffic will be untagged | ✅ |
| C) All VLAN traffic will be tagged | ❌ |
| D) All VLAN traffic will be untagged | ❌ |

> **✅ Answer: B) VLAN 44 traffic will be untagged**
> 
> - The command `switchport trunk native vlan 44` changes the native VLAN from the default (VLAN 1) to **VLAN 44**
> - Traffic in the native VLAN is **never tagged** when sent over a trunk
> - By default, VLAN 1 would be untagged, but the configuration changed it to VLAN 44
> 
> 💡 **Verification:** Use `show interfaces trunk` to see native VLAN settings.
> 

---

## 📌 Quick Review Summary

| Topic | Key Takeaway |
| --- | --- |
| **Native VLAN on Router** | Two methods: subinterface with `native` keyword OR IP on physical interface |
| **Wireshark 802.1Q** | TPID (8100), PCP, DEI, VLAN ID fields visible in tagged frames |
| **Layer 3 Switch** | Can route between VLANs using SVIs — more efficient than ROAS |
| **SVI Requirements** | VLAN exists + port up in VLAN + VLAN not shutdown + SVI not shutdown |
| **Critical Commands** | `ip routing` (global), `no switchport` (interface), `no shutdown` (SVI) |

---

## 🗺️ Concept Map

```
┌─────────────────────────────────────────────────────────────────┐
│                    INTER-VLAN ROUTING METHODS                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐    ┌──────────────────┐    ┌───────────────┐  │
│  │  Multiple    │    │  Router on a     │    │  Layer 3      │  │
│  │  Router      │    │  Stick (ROAS)    │    │  Switch (MLS) │  │
│  │  Interfaces  │    │                  │    │               │  │
│  ├──────────────┤    ├──────────────────┤    ├───────────────┤  │
│  │ • Simple     │    │ • Efficient      │    │ • Best for    │  │
│  │ • Limited    │    │   interface use  │    │   large nets  │  │
│  │   by router  │    │ • Congestion     │    │ • More complex│  │
│  │   interfaces │    │   possible       │    │   config      │  │
│  └──────────────┘    └──────────────────┘    └───────────────┘  │
│                                                                 │
│  KEY: Layer 3 Switch routes internally — traffic never goes     │
│       to the router for inter-VLAN routing!                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔑 SVI Up/UP Checklist

```
┌─────────────────────────────────────────────────────────────┐
│              SVI UP/UP REQUIREMENTS CHECKLIST               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ☐ 1. VLAN exists on the switch                             │
│       (switch won't auto-create VLAN for SVI)               │
│                                                             │
│  ☐ 2. At least one access port in VLAN is up/up             │
│       OR a trunk port allowing that VLAN is up/up           │
│                                                              │
│  ☐ 3. VLAN is not shutdown                                  │
│       (no shutdown on VLAN)                                 │
│                                                              │
│  ☐ 4. SVI is not shutdown                                   │
│       (no shutdown on SVI interface)                        │
│                                                              │
│  ✅ ALL FOUR must be true for SVI to be UP/UP               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```