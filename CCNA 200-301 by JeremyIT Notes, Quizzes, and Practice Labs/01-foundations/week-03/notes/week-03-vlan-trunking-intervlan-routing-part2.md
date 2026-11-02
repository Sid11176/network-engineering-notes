# 🔗 VLANs Part 2: VLAN Trunking & Inter-VLAN Routing

> **Course:** CCNA 200-301 | **Source:** Jeremy's IT Lab
**Tags:** #CCNA #Networking #VLANs #Trunking #ROAS #802.1Q
> 

---

## 📖 1. Core Concepts

### 🔗 What is a Trunk Port?

| Port Type | VLAN Membership | Typical Use |
| --- | --- | --- |
| **Access Port** | Belongs to a **single VLAN** | Connects to end hosts (PCs, printers) |
| **Trunk Port** | Carries traffic for **multiple VLANs** | Connects switch-to-switch or switch-to-router |

> A **trunk port** is a point-to-point link that carries traffic for **multiple VLANs** over a **single physical interface**.
> 

### 🎯 Purpose of Trunk Ports

| Benefit | Emoji | Description |
| --- | --- | --- |
| **Efficiency** | ⚡ | Reduces need for multiple physical links (one per VLAN) |
| **Scalability** | 📈 | Easily extend VLANs across a network without additional cabling |
| **Inter-Switch Communication** | 🔄 | Allows switches to share VLAN information and forward frames correctly |

---

### 🏷 802.1Q Encapsulation (The "Tag")

To identify which VLAN a frame belongs to, switches use the **IEEE 802.1Q** standard.

**How it works:**

- A **4-byte tag** is inserted into the Ethernet frame
- **Key Field:** The **VLAN ID (VID)** inside the tag (**12 bits**) specifies the exact VLAN (e.g., VLAN 10, 20, 30)
- **Result:** The receiving switch reads the VID and forwards the frame **only** to ports in that VLAN

```
┌─────────────────────────────────────────────────────────────┐
│  Ethernet Frame with 802.1Q Tag                             │
├──────────┬──────────┬──────────┬──────────┬─────────────────┤
│ Dest MAC │ Src MAC  │ 802.1Q   │ Type/    │   Data/Payload  │
│ (6 bytes)│ (6 bytes)│ Tag      │ Length   │                 │
│          │          │ (4 bytes)│ (2 bytes)│                 │
├──────────┴──────────┴──────────┴──────────┴─────────────────┤
│                    ▲                                        │
│                    │                                        │
│         ┌──────────┴────────────────────────                │
│         │ TPID  │ PCP │ DEI │ VID (12 bits) │               │
│         │(16bit)│(3b) │(1b) │               │               │
│         └───────────────────────────────────                │
└─────────────────────────────────────────────────────────────┘
```

| Field | Bits | Purpose |
| --- | --- | --- |
| **TPID** | 16 | Tag Protocol Identifier (0x8100 = 802.1Q) |
| **PCP** | 3 | Priority Code Point (QoS) |
| **DEI** | 1 | Drop Eligible Indicator |
| **VID** | 12 | **VLAN Identifier** (1–4094) |

---

### 🏠 The Native VLAN (The "Untagged" Traffic)

This is a crucial 802.1Q feature.

- **What it is:** Traffic from the **Native VLAN** is sent across the trunk **untagged** (no 802.1Q tag inserted)
- **Why it matters:** Ensures compatibility with older devices that don't understand VLAN tags
- **Default:** VLAN 1 is the Native VLAN by default

> ⚠️ **Best Practice:** The Native VLAN **must match on both ends** of the trunk. If it doesn't, traffic will be **misdirected**, causing a **VLAN mismatch** and potential security vulnerabilities.
> 

> 🛡️ **Security Note:** Change the Native VLAN from VLAN 1 to an unused, dedicated VLAN to mitigate **VLAN hopping** attacks.
> 

---

## ⌨ 2. Trunking Command Cheat Sheet (Cisco IOS)

### 🛠 Configuration Commands (Interface Config Mode)

| Command | Purpose |
| --- | --- |
| `switchport mode trunk` | **Forces** the port into permanent trunk mode |
| `switchport trunk encapsulation dot1q` | Sets trunking encapsulation to **802.1Q** (required on some switch models **before** entering trunk mode) |
| `switchport trunk native vlan [vlan-id]` | Sets the Native VLAN for the trunk (e.g., `switchport trunk native vlan 10` makes VLAN 10 the untagged VLAN) |
| `switchport trunk allowed vlan [vlan-list]` | Sets the **exact list** of VLANs allowed on the trunk (**overwrites** any previous list). Example: `switchport trunk allowed vlan 10,20,30` |
| `switchport trunk allowed vlan add [vlan-list]` | **Adds** VLANs to the existing allowed list **without overwriting** it |
| `switchport trunk allowed vlan remove [vlan-list]` | **Removes** specific VLANs from the allowed list |
| `no switchport trunk allowed vlan` | **Resets** the allowed list to the default (**all VLANs, 1–4094**) |
| `switchport nonegotiate` | Disables **DTP** (Dynamic Trunking Protocol) negotiation — a security best practice |

### 🔍 Verification Commands (Privileged EXEC Mode)

| Command | What It Shows |
| --- | --- |
| `show interfaces trunk` | **The most important command.** Displays all active trunks, their mode, encapsulation, **Native VLAN**, and the lists of **allowed and active VLANs** |
| `show interfaces [interface-id] switchport` | Provides detailed administrative and operational status of a specific port, including trunking mode and encapsulation |
| `show vlan brief` | Lists all VLANs and their associated access ports |
| `show ip interface brief` | (For ROAS) Verifies IP addresses and status of router sub-interfaces |

---

### 🧠 Pro-Tip: Understanding `show interfaces trunk`

The output has **three critical VLAN lists**:

| List | Meaning | Troubleshooting |
| --- | --- | --- |
| **Vlans allowed on trunk** | VLANs configured to pass | If a VLAN is missing here → configuration error (e.g., forgot `add` keyword) |
| **Vlans allowed and active in management domain** | VLANs that are allowed **AND exist** in the switch's VLAN database | If allowed but missing here → VLAN hasn't been **created** on the switch |
| **Vlans in spanning tree forwarding state** | VLANs actually forwarding traffic after **STP** calculations | If missing here → STP is blocking that VLAN |

---

## 🏢 3. Router on a Stick (ROAS)

> **ROAS** is a technique to enable **inter-VLAN routing** using a **single physical router interface**.
> 

### 🔄 How It Works

- The physical router interface is divided into multiple **logical sub-interfaces**, one for each VLAN
- **The Process:**
    1. The switch port connected to the router is configured as a **trunk**
    2. Each router **sub-interface** is assigned an **IP address** (acting as the default gateway for that VLAN) and is configured with **802.1Q encapsulation** for its specific VLAN ID

```
┌──────────────────────────────────────────────────────────────────────┐
│                        ROAS Topology                                 │
│                                                                      │
│  ┌─────────┐         ┌──────────────┐         ┌──────────────────┐   │
│  │  VLAN10 │         │              │         │  Router          │   │
│  │  Hosts  ├─────────┤   Switch     ├─────────┤  G0/0 (Trunk)    │   │
│  │         │         │              │         │  ├─ G0/0.10 (VLAN10) │
│  └─────────┘         │  Trunk Port  │         │  └─ G0/0.20 (VLAN20) │
│                      │              │         │                  │   │
│  ┌─────────┐         │              │         │  Each sub-interface  │
│  │  VLAN20 │         │              │         │  = default gateway   │
│  │  Hosts  ├─────────┤              │         │  for its VLAN        │
│  └─────────┘         └──────────────┘         └──────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
```

### ⚙ ROAS Configuration on the Router

```
Router> enable
Router# configure terminal

! Create sub-interface for VLAN 10
Router(config)# interface gigabitethernet 0/0.10
Router(config-subif)# encapsulation dot1q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit

! Create sub-interface for VLAN 20
Router(config)# interface gigabitethernet 0/0.20
Router(config-subif)# encapsulation dot1q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit

! Don't forget to enable the physical interface!
Router(config)# interface gigabitethernet 0/0
Router(config-if)# no shutdown
```

> 📝 **Note:** The sub-interface number does **not** have to match the VLAN ID, but it is a **best practice** to do so for clarity.
> 

### 🔍 ROAS Verification Commands

```
show ip interface brief
show running-config interface gigabitethernet 0/0.10
show vlans
```

### 🧰 Additional Useful ROAS Syntaxes (Bonus for CCNA)

| Command | Mode | Purpose |
| --- | --- | --- |
| `encapsulation dot1q [vlan-id] native` | Sub-interface Config | Sets the sub-interface as the Native VLAN (untagged) |
| `encapsulation dot1q [vlan-id] second-dot1q [vlan-id]` | Sub-interface Config | Configure **QinQ** (double tagging) for service providers |
| `show ip route` | Privileged EXEC | Verify inter-VLAN routing is working (connected routes appear) |
| `show arp` | Privileged EXEC | Verify ARP entries for hosts in different VLANs |

---

## 🛡 4. Security Best Practices

| Best Practice | Command | Why |
| --- | --- | --- |
| **Change the Native VLAN** | `switchport trunk native vlan [vlan-id]` | Do not use VLAN 1 as Native VLAN. Use an unused, dedicated VLAN to mitigate **VLAN hopping** attacks |
| **Prune Allowed VLANs** | `switchport trunk allowed vlan [vlan-list]` | Instead of allowing all VLANs, explicitly list only the VLANs that need to traverse the trunk |
| **Disable DTP** | `switchport nonegotiate` | Prevents an attacker from negotiating a trunk |
| **Tag the Native VLAN** | `vlan dot1q tag native` (global config) | Forces the switch to tag even the Native VLAN traffic and **drop any untagged frames**, preventing VLAN hopping |

> 🔐 **VLAN Hopping Attack:** An attacker can double-tag frames to jump from one VLAN to another. Mitigations include changing Native VLAN, disabling DTP, and tagging the Native VLAN.
> 

### 🧰 Additional Security Syntaxes (Bonus for CCNA)

| Command | Mode | Purpose |
| --- | --- | --- |
| `switchport mode access` | Interface Config | Force port to access mode (prevent trunk negotiation) |
| `switchport mode dynamic auto` | Interface Config | DTP mode: will trunk if the other side initiates |
| `switchport mode dynamic desirable` | Interface Config | DTP mode: actively tries to form a trunk |
| `switchport nonegotiate` | Interface Config | Disable DTP on the interface |
| `show dtp interface [interface-id]` | Privileged EXEC | Verify DTP status on an interface |

---

## ✍ 5. Quiz Answers & Explanations

### ❓ Q1: Configure SW1 to send VLAN10 frames untagged over its GigabitEthernet0/1 interface.

| Option | Answer |
| --- | --- |
| a) `switchport trunk allowed vlan 10` | ❌ |
| b) `switchport mode access` | ❌ |
| c) `switchport trunk encapsulation dot1q` | ❌ |
| d) `switchport trunk native vlan 10` | ✅ |

> **✅ Answer: d) `switchport trunk native vlan 10`**
> 
> 
> **Why:** Frames from the **Native VLAN** are sent **untagged** across an 802.1Q trunk by default.
> 

---

### ❓ Q2: After modifying the allowed VLAN list, return it to the default state.

| Option | Answer |
| --- | --- |
| a) `switchport trunk allowed vlan 1-4094` | ❌ |
| b) `switchport trunk allowed vlan all` | ✅ |
| c) `no switchport trunk allowed vlan` | ✅ (also valid) |
| d) `switchport trunk allowed vlan default` | ❌ |

> **✅ Answer: b) `switchport trunk allowed vlan all`** (and `no switchport trunk allowed vlan` also resets it)
> 
> 
> **Why:** The `no` form of the command resets the list, and `all` explicitly sets it to allow all VLANs (1–4094).
> 

> 📝 **Note:** Both `no switchport trunk allowed vlan` and `switchport trunk allowed vlan all` achieve the same result.
> 

---

### ❓ Q3: The command `switchport mode trunk` is rejected. What might fix it?

| Option | Answer |
| --- | --- |
| a) `switchport mode access` | ❌ |
| b) `switchport nonegotiate` | ❌ |
| c) `switchport trunk encapsulation dot1q` | ✅ |
| d) `no shutdown` | ❌ |

> **✅ Answer: c) `switchport trunk encapsulation dot1q`**
> 
> 
> **Why:** Some Cisco switches require you to specify the encapsulation type **before** the port can be placed into trunk mode.
> 

---

### ❓ Q4: Which field of the 802.1Q tag identifies the VLAN ID?

| Option | Answer |
| --- | --- |
| a) TPID | ❌ |
| b) VID | ✅ |
| c) PCP | ❌ |
| d) DEI | ❌ |

> **✅ Answer: b) VID**
> 
> 
> **Why:** The **VLAN Identifier (VID)** is the **12-bit field** that carries the VLAN number.
> 

---

### ❓ Q5: You added VLAN10 with `switchport trunk allowed vlan add 10`, but it's missing from "Vlans allowed and active." Why?

| Option | Answer |
| --- | --- |
| a) VLAN10 doesn't exist on the switch | ✅ |
| b) The trunk is down | ❌ |
| c) STP is blocking VLAN10 | ❌ |
| d) DTP is disabled | ❌ |

> **✅ Answer: a) VLAN10 doesn't exist on the switch.**
> 
> 
> **Why:** For a VLAN to be **"active,"** it must be **created** in the switch's VLAN database. The `allowed` command only permits it; the VLAN must also exist to become active.
> 

---

## 📋 Quick Reference Cheat Sheet

### 🔑 Key Terms

| Term | Definition |
| --- | --- |
| **Trunk Port** | Point-to-point link carrying multiple VLANs over one physical interface |
| **802.1Q** | IEEE standard for VLAN tagging (4-byte tag inserted into Ethernet frame) |
| **VID** | VLAN Identifier — 12-bit field in the 802.1Q tag (1–4094) |
| **Native VLAN** | VLAN whose traffic is sent **untagged** across a trunk (default: VLAN 1) |
| **ROAS** | Router on a Stick — inter-VLAN routing using sub-interfaces on a single router port |
| **DTP** | Dynamic Trunking Protocol — negotiates trunk formation (security risk if left enabled) |

### ⚡ Quick Command Reference

```
! ===== TRUNK CONFIGURATION (SWITCH) =====
interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q      ! Set encapsulation (if required)
 switchport mode trunk                      ! Force trunk mode
 switchport trunk native vlan 99            ! Set Native VLAN (best practice: not VLAN 1)
 switchport trunk allowed vlan 10,20,30     ! Restrict allowed VLANs
 switchport nonegotiate                     ! Disable DTP

! ===== TRUNK VERIFICATION =====
show interfaces trunk
show interfaces GigabitEthernet0/1 switchport
show vlan brief

! ===== ROAS CONFIGURATION (ROUTER) =====
interface GigabitEthernet0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
!
interface GigabitEthernet0/0
 no shutdown

! ===== ROAS VERIFICATION =====
show ip interface brief
show ip route

! ===== SECURITY BEST PRACTICES =====
vlan dot1q tag native                       ! Global: tag Native VLAN traffic
switchport nonegotiate                      ! Interface: disable DTP
switchport trunk native vlan 99             ! Change Native VLAN from VLAN 1
switchport trunk allowed vlan 10,20,30      ! Prune unnecessary VLANs
```

---

## 🧠 Key Takeaways

1. **Trunk ports** carry multiple VLANs over a single link — essential for scalable networks
2. **802.1Q** uses a **4-byte tag** with a **12-bit VID** to identify VLAN membership
3. **Native VLAN** traffic is sent **untagged** — must match on both ends of the trunk
4. **ROAS** uses **sub-interfaces** with 802.1Q encapsulation for inter-VLAN routing
5. **Security:** Change Native VLAN, prune allowed VLANs, disable DTP, and tag Native VLAN
6. **`show interfaces trunk`** is your best friend — check the **three VLAN lists** to troubleshoot