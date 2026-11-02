# 🛡️ BPDU Guard & BPDU Filter - CCNA Notes

## 📚 Overview

Continuing the STP Toolkit section. These features are often used with **PortFast** on ports connected to end hosts.

**Topics Covered:**

- BPDU Guard - Disables port if BPDU received
- BPDU Filter - Stops port from sending/processing BPDUs

> ⚠️ **Note:** Root Guard and Loop Guard will be covered in the next video
> 

---

## 🔄 PortFast Review

**What PortFast Does:**

- Port starts in **Forwarding state** immediately upon connection
- Does **NOT** disable STP on the port

**Key Behavior:**

| Action | Status |
| --- | --- |
| Sends BPDUs | ✅ Yes (every 2 seconds) |
| Receives BPDUs from PC | ❌ No (end hosts don't run STP) |

**What Happens if PortFast Port Receives BPDU?**

- Port reverts to acting like a **regular STP port** (PortFast disabled)
- Can cause problems if unauthorized switch is connected!

---

## 😱 The Problem: Unauthorized Switches

### Scenario: "Bob from Accounting"

- Bob brings personal switch to work
- Connects to wall jack (intended for end hosts)
- Bob's switch starts exchanging BPDUs with company switch

### Potential Issues:

1. **Root Bridge Hijacking** - Bob's switch could become root bridge (lower bridge ID)
2. **Suboptimal Paths** - Traffic flows through less efficient routes
3. **Network Instability** - Unauthorized device affects STP topology

```
Normal: SW1 (Root) ← SW2 ← SW3
After Bob: SW1 ← SW2 ← SW3 ← Bob's Switch (Root!)
```

---

## 🛡️ BPDU Guard

### What It Does

- **Protects network** from unauthorized switches on end-host ports
- If BPDU received → port enters **error-disabled state**
- Port can no longer send or receive data/BPDUs

### Port Behavior

| Action | BPDU Guard Enabled |
| --- | --- |
| Sends BPDUs | ✅ Yes |
| Receives BPDU | ❌ Port error-disabled |

### Configuration Syntax

**Per-Port Configuration:**

```
interface GigabitEthernet0/1
 spanning-tree bpduguard enable
```

**Global Default Configuration:**

```
spanning-tree portfast bpduguard default
```

> 💡 Optionally add `edge` keyword in modern IOS
> 

**Disable on Specific Port:**

```
interface GigabitEthernet0/1
 spanning-tree bpduguard disable
```

### ⚠️ Important Distinction

- **PortFast default** → activates on **all access ports**
- **BPDU Guard default** → activates on **all PortFast-enabled ports**
- ⭐ This difference CAN be tested on CCNA exam!

### Verification Commands

```
show spanning-tree interface GigabitEthernet0/1 detail
```

### Console Messages When Triggered

```
%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU on port GigabitEthernet0/1 with BPDU Guard enabled. Disabling port.
%PM-4-ERR_DISABLE: bpduguard error detected on Gi0/1, putting Gi0/1 in err-disable state
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down
```

### Visual Indicator

- Link light turns **solid amber** when err-disabled
- Remains amber even after cable disconnect

---

## 🔧 ErrDisable Recovery

### What Is It?

Feature that **automatically re-enables** err-disabled ports after a period of time

### Default Settings

| Setting | Value |
| --- | --- |
| ErrDisable Recovery | ❌ Disabled by default |
| Recovery Timer | 300 seconds (5 minutes) |

### Configuration Syntax

**View Status:**

```
show errdisable recovery
```

**Enable for BPDU Guard:**

```
errdisable recovery cause bpduguard
```

**Modify Timer (optional):**

```
errdisable recovery interval <seconds>
```

### Two Ways to Re-enable Err-Disabled Ports

**1. Manual:**

```
interface GigabitEthernet0/1
 shutdown
 no shutdown
```

**2. Automatic (ErrDisable Recovery):**

- Enable with `errdisable recovery cause bpduguard`
- Port auto-recovers after timer expires

### ⚠️ Critical Point

> **Always fix the underlying issue first!** If Bob's switch is still connected, the port will just be err-disabled again immediately after recovery.
> 

### Other ErrDisable Causes (CCNA Topics)

- `arp-inspection` (DAI - Dynamic ARP Inspection)
- `psecure-violation` (Port Security)
- `policing` (Power Policing)

---

## 🔇 BPDU Filter

### The Problem It Solves

- Switch port sends BPDUs every 2 seconds to end hosts
- **Unnecessary** - end hosts ignore BPDUs
- **Security concern** - BPDUs contain STP topology info

### What It Does

- **Prevents port from sending BPDUs**
- Does **NOT** disable port if BPDU received
- Behavior depends on **how it's configured**

### Two Configuration Methods (Different Behavior!)

#### Method 1: Per-Port Configuration

```
interface GigabitEthernet0/1
 spanning-tree bpdufilter enable
```

**Behavior:**

| Action | Result |
| --- | --- |
| Sends BPDUs | ❌ No |
| Receives BPDU | ❌ Ignored (STP effectively disabled on port) |

> 🚨 **DANGER!** This can cause **permanent Layer-2 loops** if enabled on wrong port!
> 

#### Method 2: Global Default Configuration (Recommended ✅)

```
spanning-tree portfast bpdufilter default
```

> 💡 Optionally add `edge` keyword after portfast
> 

**Behavior:**

| Action | Result |
| --- | --- |
| Sends BPDUs | ❌ No |
| Receives BPDU | ✅ PortFast & BPDU Filter disabled, port operates as normal STP port |

**This is the best of both worlds!**

**Disable on Specific Port:**

```
interface GigabitEthernet0/1
 spanning-tree bpdufilter disable
```

---

## 🎯 BPDU Guard + BPDU Filter Together

### Can They Be Enabled Simultaneously?

**Yes**, but behavior depends on BPDU Filter configuration:

| BPDU Filter Method | BPDU Received | Result |
| --- | --- | --- |
| **Global default** | Received | BPDU Filter disabled → BPDU Guard triggered → Port err-disabled |
| **Per-port** | Received | BPDU ignored → BPDU Guard NOT triggered |

> 💡 **Recommendation:** Use BPDU Filter in **global config mode** only!
> 

---

## 📋 Quick Reference: Configuration Syntax

### BPDU Guard

```
! Per-port
interface GigabitEthernet0/1
 spanning-tree bpduguard enable

! Global default
spanning-tree portfast bpduguard default

! Disable on specific port
interface GigabitEthernet0/1
 spanning-tree bpduguard disable
```

### BPDU Filter

```
! Per-port (⚠️ Use with caution!)
interface GigabitEthernet0/1
 spanning-tree bpdufilter enable

! Global default (✅ Recommended)
spanning-tree portfast bpdufilter default

! Disable on specific port
interface GigabitEthernet0/1
 spanning-tree bpdufilter disable
```

### ErrDisable Recovery

```
! View status
show errdisable recovery

! Enable for BPDU Guard
errdisable recovery cause bpduguard

! Modify timer
errdisable recovery interval <seconds>
```

### Verification

```
show spanning-tree interface GigabitEthernet0/1 detail
show interfaces GigabitEthernet0/1
show errdisable recovery
```

---

## 🔑 Key Takeaways

| Feature | Sends BPDU? | On BPDU Received |
| --- | --- | --- |
| **PortFast** | ✅ Yes | Reverts to normal STP port |
| **BPDU Guard** | ✅ Yes | **Err-disables port** |
| **BPDU Filter (per-port)** | ❌ No | **Ignores BPDU** (dangerous!) |
| **BPDU Filter (global)** | ❌ No | Disables PortFast/Filter, normal STP |

### Best Practices

1. ✅ Enable PortFast on end-host ports
2. ✅ Enable BPDU Guard to protect against unauthorized switches
3. ✅ Use BPDU Filter **global mode only** (if needed)
4. ⚠️ Never enable BPDU Filter per-port on switch-to-switch links
5. 🔧 Always fix underlying issues before re-enabling err-disabled ports

---

## 📝 Exam Tips

- Know the **difference** between PortFast default (all access ports) vs BPDU Guard default (all PortFast ports)
- Understand **two behaviors** of BPDU Filter based on configuration method
- Remember: BPDU Guard = **err-disable**, BPDU Filter = **no err-disable**
- ErrDisable Recovery default timer = **300 seconds (5 minutes)**
- Know how to **manually and automatically** recover err-disabled ports