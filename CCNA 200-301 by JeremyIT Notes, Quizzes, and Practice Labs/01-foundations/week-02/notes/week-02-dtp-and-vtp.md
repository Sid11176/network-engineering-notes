# 🔌 DTP & VTP — Dynamic Trunking Protocol & VLAN Trunking Protocol

> **Course:** CCNA 200-301 | **Source:** Jeremy's IT Lab
**Tags:** #CCNA #Networking #DTP #VTP #Trunking #VLANs
> 

> ⚠️ **CCNA Note:** DTP and VTP were **removed** from the 200-301 exam topics list, but you may still get **basic questions** about their purpose/function.
> 

---

## 🔌 Part 1: DTP — Dynamic Trunking Protocol

### 📖 What It Is

> A **Cisco proprietary** Layer 2 protocol that lets Cisco switches **dynamically negotiate** whether an interface becomes an **access port** or a **trunk port** without manual configuration.
> 

### 🔑 Key Characteristics

| Feature | Detail |
| --- | --- |
| **Default state** | Enabled by default on Cisco switch interfaces |
| **Compatibility** | Only works between **Cisco devices** |
| **Security recommendation** | 🚨 **Disable DTP** and manually configure access/trunk ports |
| **Router/PC behavior** | DTP will **NOT** form a trunk with a router/PC → manually configure trunk for router-on-a-stick |
| **Access mode** | Disables DTP negotiation |
| **Manual trunk mode** | **Still sends DTP frames** unless you also use `switchport nonegotiate` |

---

### 🎛 DTP Administrative Modes

| Mode | Command | Behavior |
| --- | --- | --- |
| **Static Access** | `switchport mode access` | Static access; DTP disabled |
| **Manual Trunk** | `switchport mode trunk` | Manual trunk; **still sends DTP** unless `nonegotiate` |
| **Dynamic Desirable** | `switchport mode dynamic desirable` | **Actively** tries to form a trunk |
| **Dynamic Auto** | `switchport mode dynamic auto` | **Passive**; forms trunk only if other side actively tries |
| **Nonegotiate** | `switchport nonegotiate` | Stop sending DTP frames |

### 🏭 Default Modes by Switch Generation

| Switch Generation | Default Mode |
| --- | --- |
| **Older switches** | `dynamic desirable` |
| **Newer switches** | `dynamic auto` |

> 💡 **Both new switches default to auto + auto → ACCESS, not trunk.**
> 

---

### 📊 DTP Operational Result Matrix

| Local \ Peer | Access | Dynamic Auto | Dynamic Desirable | Trunk |
| --- | --- | --- | --- | --- |
| **Access** | Access | Access | Access | ⚠️ **Misconfig** |
| **Dynamic Auto** | Access | **Access** | **Trunk** | **Trunk** |
| **Dynamic Desirable** | Access | **Trunk** | **Trunk** | **Trunk** |
| **Trunk** | ⚠️ **Misconfig** | **Trunk** | **Trunk** | **Trunk** |

> ⚠️ **Misconfig Warning:** Access + Trunk / Trunk + Access = **misconfiguration**; traffic will **not pass correctly**. Do **not** use in production.
> 

---

### 🏷 Trunk Encapsulation Negotiation

- Switches supporting both **dot1q** and **ISL** can negotiate encapsulation via DTP.
- **Default:** `switchport trunk encapsulation negotiate`
- To manually configure a trunk on a switch supporting both, you **must set encapsulation first** — you **cannot** leave it as `negotiate`.
- If both support ISL, **ISL is preferred** over dot1q.

### 📡 DTP Frame VLANs

| Encapsulation | DTP Frames Sent In |
| --- | --- |
| **ISL** | VLAN 1 |
| **dot1q** | Native VLAN (default = VLAN 1) |

### 🔍 `show interfaces switchport` Field: "Negotiation of Trunking"

| State | Result |
| --- | --- |
| `dynamic desirable`, `dynamic auto`, or `trunk` | **On** |
| `access` mode OR `switchport nonegotiate` | **Off** |

---

### 🛠 DTP Commands & Modes

| Command | Mode | Purpose |
| --- | --- | --- |
| `switchport mode access` | Interface config | Static access; disables DTP |
| `switchport mode trunk` | Interface config | Manual trunk; still sends DTP |
| `switchport mode dynamic desirable` | Interface config | Actively negotiate trunk |
| `switchport mode dynamic auto` | Interface config | Passively accept trunk |
| `switchport nonegotiate` | Interface config | Stop sending DTP frames |
| `switchport trunk encapsulation negotiate` | Interface config | Default encapsulation negotiation |
| `switchport trunk encapsulation dot1q` | Interface config | Set dot1q encapsulation |
| `switchport trunk encapsulation isl` | Interface config | Set ISL encapsulation |
| `show interfaces G0/0 switchport` | EXEC / privileged EXEC | Show admin/operational mode, DTP negotiation, encapsulation |
| `show interfaces switchport` | EXEC / privileged EXEC | Same for all interfaces |

---

## 🧪 Part 2: VTP — VLAN Trunking Protocol

### 📖 What It Is

> A **Cisco proprietary** protocol that lets you configure VLANs on a central **VTP server**; **VTP clients** synchronize their VLAN database to the server.
> 

### 🔑 Key Characteristics

| Feature | Detail |
| --- | --- |
| **Designed for** | Large networks with many VLANs |
| **Modern recommendation** | 🚨 Rarely used today; generally recommended **NOT** to use it |
| **Scope** | Syncs **only the VLAN database** — does **NOT** assign interfaces to VLANs |
| **Interface assignment** | You still configure `switchport access vlan X` on each switch manually |
| **Advertisements** | Sent **only on trunk ports** |

---

### 🔁 VTP Revision Number

> The **highest revision number = newest VLAN database**.
> 
- Revision number **increases** when a VLAN is added, modified, or deleted.
- **VTP servers also act as clients:** they will sync to another server with a **higher revision number**.

#### 🚨 The Danger

> Connecting an old switch with a **higher revision number** and **matching domain** can **overwrite the network VLAN database**, causing **outages**.
> 

#### 🔄 How to Reset Revision Number to 0

1. ✅ Change VTP domain to an **unused name**.
2. ✅ Change VTP mode to **transparent**.
3. ❌ `vtp reset` is **NOT** a valid method.

---

### 🧩 VTP Modes

| Mode | Add/Modify/Delete VLANs? | Stores VLAN DB in NVRAM? | Syncs to Server? | Advertises/Forwards? |
| --- | --- | --- | --- | --- |
| **Server** | ✅ Yes | ✅ Yes | ✅ Yes, if higher revision received | ✅ Yes |
| **Client** | ❌ No | ❌ No (except VTPv3) | ✅ Yes | ✅ Yes, local only |
| **Transparent** | ✅ Yes | ✅ Yes | ❌ No | ➡️ Forwards same-domain advertisements; **does not advertise its own DB** |

> 💡 **Default Cisco switch mode:** **VTP Server**
💡 **Default VTP domain:** **NULL**
> 
> - If a switch with NULL/no domain receives an advertisement with a domain name, it **automatically joins** that domain.

> 📌 **Default existing VLANs:** **5** → VLAN 1 and 1002–1005.
> 

---

### 🧾 VTP Versions

| Version | Notes |
| --- | --- |
| **VTPv1** | Default version |
| **VTPv2** | Adds Token Ring VLAN support; otherwise little difference |
| **VTPv3** | Supports extended VLAN range **1006–4094**; VTPv3 clients **store VLAN DB in NVRAM** |

> ⚠️ **VTPv1/v2** support only VLANs **up to 1005** locally.
⚠️ Changing VTP version **increases the revision number**.
> 

---

### 🛠 VTP Commands & Modes

| Command | Mode | Purpose |
| --- | --- | --- |
| `vtp domain CISCO` | Global config | Set VTP domain name |
| `vtp mode server` | Global config | Set VTP server mode |
| `vtp mode client` | Global config | Set VTP client mode |
| `vtp mode transparent` | Global config | Set VTP transparent mode |
| `vtp version 2` | Global config | Change VTP version; increases revision |
| `vlan 10` | Global config | Create VLAN; enters VLAN config |
| `name engineering` | VLAN config | Name the VLAN |
| `show vtp status` | EXEC / privileged EXEC | Show domain, mode, version, revision, VLAN counts |
| `show vlan brief` | EXEC / privileged EXEC | Show VLAN database |

---

### 🧰 Additional Useful Syntaxes (Bonus for CCNA)

| Command | Mode | Purpose |
| --- | --- | --- |
| `show vtp password` | Privileged EXEC | Display VTP password (if configured) |
| `vtp password [password]` | Global config | Set VTP password for authentication |
| `vtp pruning` | Global config | Enable VTP pruning (reduce unnecessary flooded traffic) |
| `show interfaces trunk` | Privileged EXEC | Verify trunk ports and allowed VLANs |
| `switchport trunk allowed vlan [list]` | Interface config | Control which VLANs traverse a trunk |
| `switchport trunk native vlan [id]` | Interface config | Change the native VLAN on a trunk |
| `show dtp interface [type]` | Privileged EXEC | Display DTP information for an interface |
| `debug dtp [events|packets]` | Privileged EXEC | Troubleshoot DTP negotiations (use with caution) |
| `show spanning-tree interface [type]` | Privileged EXEC | Verify STP state post-DTP negotiation |

---

## ❓ End-of-Lesson Quiz

### 📝 Q1: Old Switch Forms Unexpected Trunk

**Scenario:** SW1 and SW2 are connected, both new switches, interfaces operating as access ports. SW2's power supply fails, so you replace it with an old spare switch. You reset the old switch's configuration before connecting it, but a **trunk forms** between SW1 and the old switch. What could be the cause?

| Option | Answer |
| --- | --- |
| A. Interfaces on old switches default to `switchport mode trunk` | ❌ |
| B. Interfaces on old switches default to `switchport mode dynamic desirable` | ✅ |
| C. Access ports are a feature of newer switches | ❌ |

> **✅ Answer: BExplanation:** Old switches default to `dynamic desirable`; newer switches default to `dynamic auto`. **Auto + desirable = trunk**.
> 

---

### 📝 Q2: Forward but Don't Sync — VTP Mode

**Scenario:** SW1 is connected to SW2, and SW2 is connected to SW3. You want SW2 to **forward** SW1's VLAN database information to SW3, but you do **not** want SW2 to **synchronize** its VLAN database to SW1. Which command should you use on SW2?

| Option | Answer |
| --- | --- |
| A. `vtp mode transparent` | ✅ |
| B. `vtp transparent mode` | ❌ |
| C. `vlan mode transparent` | ❌ |
| D. `vtp mode client` | ❌ |

> **✅ Answer: AExplanation:** `vtp mode transparent` **forwards VTP advertisements** but does **not sync** its VLAN database to the server and does **not advertise** its own VLAN database.
> 

---

### 📝 Q3: Reset VTP Revision Number to 0 (Choose Two)

**What are two methods to reset a switch's VTP revision number to 0?**

| Option | Answer |
| --- | --- |
| A. Change the VTP domain to an unused domain name | ✅ |
| B. Change the switch to VTP server mode | ❌ |
| C. Change the switch to VTP transparent mode | ✅ |
| D. Use the `vtp reset` command | ❌ |

> **✅ Answers: A and CExplanation:** Changing to an **unused domain** or changing to **transparent mode** resets the revision number to 0. `vtp reset` is **not** a valid method.
> 

---

### 🧠 Boson ExSim Drag-and-Drop: DTP Operational Modes

| Local | Peer | Result |
| --- | --- | --- |
| Access | Access | **Access** |
| Access | Dynamic Auto | **Access** |
| Access | Dynamic Desirable | **Access** |
| Access | Trunk | ⚠️ **Misconfig** |
| Dynamic Auto | Dynamic Auto | **Access** |
| Dynamic Auto | Dynamic Desirable | **Trunk** |
| Dynamic Auto | Trunk | **Trunk** |
| Dynamic Desirable | Dynamic Desirable | **Trunk** |
| Dynamic Desirable | Trunk | **Trunk** |
| Trunk | Trunk | **Trunk** |
| Trunk | Access | ⚠️ **Misconfig** |

---

## 📋 Quick Reference Cheat Sheet

### 🔑 Key Takeaways

> ✅ **DTP negotiates access/trunk status.**
✅ **VTP synchronizes VLAN databases.**
✅ Know their **modes**, **defaults**, **revision-number behavior**, and **when to disable/avoid them**.
> 

### 🚨 Security & Best Practices

| Feature | Recommendation |
| --- | --- |
| **DTP** | 🔒 **Disable** (`switchport nonegotiate`) and manually configure access/trunk |
| **VTP** | ⚠️ **Avoid** on modern networks; use transparent mode if needed |
| **Router-on-a-stick** | 🛠 Manually configure switch port as **trunk** (DTP won't form trunk with router) |

### 🏭 Defaults to Memorize

| Setting | Default |
| --- | --- |
| **DTP (old switches)** | `dynamic desirable` |
| **DTP (new switches)** | `dynamic auto` |
| **VTP mode** | `server` |
| **VTP domain** | NULL (no domain) |
| **VTP version** | v1 |
| **Trunk encapsulation** | `negotiate` |
| **Native VLAN (dot1q)** | VLAN 1 |
| **Default VLANs** | 5 (VLAN 1 + 1002–1005) |