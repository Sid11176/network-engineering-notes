# 📚 CCNA Day 23: EtherChannel

## 🎯 Overview

EtherChannel groups multiple physical interfaces into a single logical interface, solving STP blocking issues and providing redundancy + increased bandwidth.

---

## 🔴 The Problem: STP Blocks Redundant Links

**Scenario:** Multiple links between two switches

- **Without EtherChannel:** STP disables all but one link (orange lights) to prevent Layer 2 loops
- **Result:** Wasted bandwidth, only 1 link forwarding traffic
- **With EtherChannel:** All links act as ONE logical interface, all forwarding (green lights)

```
Without EtherChannel:          With EtherChannel:
[SW1]---[SW2] ✅              [SW1]=====[SW2] ✅
[SW1]---[SW2] ❌ (blocked)    (4 links = 1 logical)
[SW1]---[SW2] ❌ (blocked)
[SW1]---[SW2] ❌ (blocked)
```

---

## 📖 Key Concepts

| Term | Description |
| --- | --- |
| **EtherChannel** | Group of physical interfaces acting as one logical interface |
| **Port-Channel** | Cisco term for the virtual interface created |
| **LAG** | Link Aggregation Group (vendor-neutral term) |
| **Oversubscription** | When end-host bandwidth > uplink bandwidth |

**Benefits:**

- ✅ Increased bandwidth (combined links)
- ✅ Redundancy (if one link fails, others continue)
- ✅ STP treats it as single interface (no blocked ports)

---

## ⚖️ Load Balancing

EtherChannel load balances based on **flows** (communication between two nodes).

**Key principle:** Same flow = same physical interface (prevents out-of-order frames)

**Load-balancing methods:**

| Method | Input Used |
| --- | --- |
| Source MAC | All frames from same source use same link |
| Destination MAC | All frames to same dest use same link |
| Source + Dest MAC | Based on both |
| Source IP | All frames from same IP use same link |
| Destination IP | All frames to same IP use same link |
| Source + Dest IP | Based on both |
| Layer 4 ports | TCP/UDP port numbers (some switches) |

---

## 🔧 Configuration Methods

### 1️⃣ PAgP (Port Aggregation Protocol)

- **Cisco proprietary** (Cisco switches only)
- Dynamically negotiates EtherChannel

| Mode | Behavior |
| --- | --- |
| **Desirable** | Actively tries to form EtherChannel |
| **Auto** | Passive, waits for other side |

**Valid combinations:**

- ✅ Desirable + Desirable
- ✅ Desirable + Auto
- ❌ Auto + Auto (no EtherChannel formed)

### 2️⃣ LACP (Link Aggregation Control Protocol)

- **IEEE 802.3ad** (industry standard, multi-vendor)
- **Preferred method** for CCNA exam

| Mode | Behavior |
| --- | --- |
| **Active** | Actively tries to form EtherChannel |
| **Passive** | Passive, waits for other side |

**Valid combinations:**

- ✅ Active + Active
- ✅ Active + Passive
- ❌ Passive + Passive (no EtherChannel formed)

### 3️⃣ Static EtherChannel

- No negotiation protocol
- Manually configured with `on` mode
- ⚠️ Not recommended (no dynamic maintenance)

**Valid combination:**

- ✅ On + On only

**Limit: Up to 8 active interfaces per EtherChannel. LACP allows up to 16, but only 8 active; the rest are standby.**

---

## ⚠️ Member Interface Requirements

All member interfaces **MUST** match:

- Same duplex
- Same speed
- Same switchport mode (access/trunk)
- Same allowed VLANs (if trunk)
- Same native VLAN (if trunk)

❌ Mismatched interfaces are **excluded** (suspended)

---

## 📊 Verification Commands Summary

| Flag | Meaning |
| --- | --- |
| **SU** | Layer 2 EtherChannel, In Use |
| **RU** | Layer 3 EtherChannel, In Use |
| **P** | Port is bundled (good!) |
| **D** | Down |
| **s** | Suspended (config mismatch) |

---

## 🔀 Layer 3 EtherChannel

Modern design trend: Use routed ports between switches (no STP needed)

**Key points:**

- Configure `no switchport` on member interfaces
- IP address goes on **port-channel interface** (not physical)
- Flag shows **R** (routed) instead of **S**
- No STP on Layer 3 connections

---

## 📝 Command Syntax Reference

### 🔍 Verification Commands

| Command | Mode | Description |
| --- | --- | --- |
| `show etherchannel load-balance` | Privileged EXEC | Display current load-balancing method |
| `show etherchannel summary` | Privileged EXEC | Display all EtherChannels and status flags |
| `show etherchannel port-channel` | Privileged EXEC | Display detailed port-channel info (including mode) |
| `show interfaces trunk` | Privileged EXEC | Verify trunk (shows Po interface, not physical) |
| `show ip interface brief` | Privileged EXEC | Verify IP on port-channel (L3) |
| `show running-config` | Privileged EXEC | Verify configurations |

### ⚙️ Configuration Commands

| Command | Mode | Description |
| --- | --- | --- |
| `port-channel load-balance [method]` | Global Config | Set load-balancing method |
| `interface range [range]` | Global Config | Enter interface range config mode |
| `channel-group [number] mode [mode]` | Interface Config | Add interface to EtherChannel |
| `channel-protocol [lacp\|pagp]` | Interface Config | Manually set negotiation protocol (usually unnecessary) |
| `interface port-channel [number]` | Global Config | Enter port-channel interface config |
| `no switchport` | Interface Config | Make interface Layer 3 routed |
| `ip address [ip] [mask]` | Interface Config | Assign IP (on port-channel for L3) |
| `switchport mode trunk` | Interface Config | Configure as trunk (on port-channel) |

### 📋 Channel-Group Modes

| Mode | Protocol | Description |
| --- | --- | --- |
| `desirable` | PAgP | Actively negotiate |
| `auto` | PAgP | Passively wait |
| `active` | LACP | Actively negotiate |
| `passive` | LACP | Passively wait |
| `on` | Static | No negotiation |

---

## 🧠 Quiz

### Question 1

**Which channel-group mode combinations result in an operational EtherChannel? (Choose 3)**

- A) on – on
- B) passive – passive
- C) desirable – auto
- D) auto – auto
- E) active – desirable
- F) on – desirable
- G) active – active

**A, C, G**

- **A (on–on):** Static EtherChannel, no protocol
- **C (desirable–auto):** PAgP negotiation
- **G (active–active):** LACP negotiation

❌ B: Passive–passive won't form (neither initiates)

❌ D: Auto–auto won't form (neither initiates)

❌ E, F: Mixing different protocols/modes fails

---

### Question 2

**In `show etherchannel summary`, physical interfaces have flag (P). What does this mean?**

- A) Interfaces are in LACP passive mode
- B) Interfaces are bundled in the port-channel
- C) Interfaces are paused
- D) EtherChannel is Layer 2

**B) Interfaces are bundled in the port-channel**

The **P** flag indicates the port is properly bundled — this is what you want to see!

---

### Question 3

**Which member interface parameters must match to form an EtherChannel? (Choose 2)**

- A) Interface ID
- B) IP address
- C) Interface speed
- D) Switchport mode (access/trunk)

**C, D**

- **C:** Speed must match
- **D:** Switchport mode must match

❌ A: Interface IDs are unique (can't match)

❌ B: IP goes on port-channel, not physical interfaces

---

### 🎁 Bonus Question (Boson ExSim)

**SwitchA config:**

```
interface port-channel 1
interface range FastEthernet0/5-6
 channel-protocol lacp
 channel-group 1 mode on
```

**SwitchB config:**

```
interface port-channel 1
interface range FastEthernet0/5-6
 channel-protocol pagp
 channel-group 1 mode on
```

**What is the result?**

- A) Link formed using LACP (configured first)
- B) No link is formed
- C) Link formed using PAgP (configured last)
- D) Link formed without aggregation protocol

**B) No link is formed**

**Reason:**

- `channel-protocol lacp` restricts modes to `active`/`passive`
- `channel-protocol pagp` restricts modes to `desirable`/`auto`
- `channel-group 1 mode on` is **rejected** on both switches due to protocol mismatch
- The port-channel interface exists, but no physical interfaces join it