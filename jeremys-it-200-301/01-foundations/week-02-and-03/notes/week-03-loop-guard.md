# 🌐 STP Toolkit: Loop Guard

> 🎯 **Purpose:** Loop Guard acts as an **extra line of defense against Layer 2 loops**. It protects the network from loops by **disabling a port if it unexpectedly stops receiving BPDUs**, ensuring the port does not mistakenly enter the Forwarding state.
> 

---

## 📚 Where It Fits in the STP Toolkit

The STP toolkit features covered so far:

| Feature | Role |
| --- | --- |
| ⚡ PortFast | Skips listening/learning on access ports |
| 🛡️ BPDU Guard | Disables port if BPDUs are received |
| 🚫 BPDU Filter | Stops sending/receiving BPDUs |
| 👑 Root Guard | Prevents a port from becoming a root port |
| 🔁 **Loop Guard** | **Blocks port if BPDUs stop arriving** |

---

## ⚠️ The Problem Loop Guard Solves

### 🔗 Unidirectional Links

- **Definition:** A network link where data transmission occurs in **only one direction** (e.g., SW1 can send frames to SW2, but SW2 cannot send frames to SW1).
- **Cause:** Typically a **Layer 1 issue** — a physical hardware problem:
    - 🧵 Damaged cables
    - 🔌 Faulty connectors or transceivers (e.g., **SFP** — Small Form-Factor Pluggable)
- 📊 **More common with fiber-optic cables than copper UTP.**

### 💡 Why Fiber Is More Vulnerable

- Fiber connections use **two separate fibers** (one per direction):
    - Tx of SW1 → Rx of SW2
    - Tx of SW2 → Rx of SW1
- Although separate fibers, they are considered **one cable** connecting one interface to another.
- If **one fiber is damaged**, data flow in **one direction** is disrupted while the other remains unaffected.
- Fiber is more fragile than copper UTP — bend it too far and you'll hear a *snap*; the cable is dead. 💔

### 🔍 Detection (or Lack Thereof)

- For a fiber interface to be **up/up**, **both fibers must be connected and functional**.
- ✅ If devices **detect** the problem → they disable their interfaces → link goes down entirely (no unidirectional link).
- ❌ If devices **fail to detect** the problem → interfaces remain **up/up** → **unidirectional link** exists.

---

## 🔄 How a Unidirectional Link Causes a Loop

1. 🟢 Normally, **SW3 G0/1** is a **non-designated blocking port** because it receives **superior BPDUs from SW2**.
2. 💥 The SW2–SW3 link becomes unidirectional — SW2's BPDUs can't reach SW3, but interfaces stay up/up.
3. ⏳ SW3 stops receiving BPDUs and its **max age timer expires**.
4. 🔓 SW3 G0/1 becomes a **Designated port** and starts forwarding BPDUs to SW2.
5. 🙈 SW2 **ignores** SW3's inferior BPDUs.
6. 🔁 **Both SW2 G0/1 and SW3 G0/1 are now Forwarding → Layer 2 loop!**

> 📡 **Result:** Broadcast frames flooded to SW3 loop endlessly around SW1 → SW3 → SW2. (Frames flooded to SW2 don't loop because of the unidirectional link.)
> 

> 💭 **Ideal outcome:** If the hardware issue caused the link to go down entirely, both switches would disable their ports and no loop would occur.
> 

---

## 🛡️ How Loop Guard Works

- Loop Guard **does not prevent** physical unidirectional links — it **detects** them and prevents Layer 2 loops.
- When a Loop Guard-enabled port's **max age timer counts down to 0**:
    - ❌ It does **NOT** become a designated port.
    - ✅ It enters the **broken (loop inconsistent)** state → the port is **blocked**.
- 📌 Note: The port **remains up/up** — it isn't shut down, STP just blocks it (same as Root Guard's *root inconsistent* state).
- 🔄 **Recovery is automatic:** If the physical issue is fixed and the port starts receiving BPDUs again, it is automatically re-enabled. No commands needed!

### 📟 CLI Behavior When Blocked

- Message displayed:
    
    ```
    Loop guard blocking port GigabitEthernet0/1
    ```
    
- In `show spanning-tree` output:
    - Status = **BKN** (Broken)
    - Flag = **LOOP_Inc** (Loop Inconsistent)

---

## ⚙️ Configuration

### 📍 Two Ways to Enable

| Method | Command | Scope |
| --- | --- | --- |
| Per-port | `spanning-tree guard loop` | Interface config mode |
| By default | `spanning-tree loopguard default` | Global config mode (all ports) |
- When enabled globally, disable on specific ports with:
    
    ```
    spanning-tree guard none
    ```
    

### 🎯 Where to Enable It

> ✅ Activate Loop Guard on **any ports that aren't designated: Root and Non-Designated ports** — i.e., ports that are *supposed to receive BPDUs* from designated ports.
> 

### 🔍 Verification

- `show spanning-tree interface detail` → shows **"Loop guard is enabled on the port"**
- If enabled by default → output adds **"by default"** at the end.

---

## 🚫 Loop Guard vs. Root Guard (Mutually Exclusive!)

> ⛔ **These two features cannot be enabled on the same port at the same time** — they serve **opposite purposes**.
> 

| Feature | Purpose | Trigger |
| --- | --- | --- |
| 👑 **Root Guard** | Prevents **designated ports** from becoming **root ports** | Port receives **superior BPDUs** → blocked |
| 🔁 **Loop Guard** | Prevents **non-designated/root ports** from becoming **designated ports** | Port **stops receiving BPDUs** → blocked |

### 📋 Example Placement

- SW3 **G0/0** and **G0/1** → ✅ **Loop Guard** (they should receive BPDUs from SW1 and SW2)
- SW3 **G0/2** → ✅ **Root Guard** (connects to a switch you don't control)

> ⚠️ Configuring Root Guard on G0/0 and G0/1 would block both ports and cut off communication in the LAN!
> 

### 🔄 Configuration Conflict Behavior

- Configuring **Root Guard** on a Loop Guard-enabled port → **Loop Guard is disabled** (Root Guard replaces it).
- Configuring **Loop Guard** on a Root Guard-enabled port → **Root Guard is disabled**.
- If Loop Guard is enabled **by default** (global) and you configure Root Guard on a port → **Loop Guard is disabled on that port**.
- 🧠 **Rule of thumb:** The **more specific configuration takes effect** — the interface command overrides the global command. (Same behavior applies to PortFast, BPDU Guard, and BPDU Filter.)

---

## 📝 Summary

| 🔑 Key Point | Details |
| --- | --- |
| 🎯 Purpose | Block a port if it unexpectedly stops receiving BPDUs |
| 🔗 Common Cause | Unidirectional link (usually Layer 1 fiber issues) or software bug |
| 🚫 Action | Port enters **broken (loop inconsistent)** state |
| 🔄 Recovery | Automatic when BPDUs are received again |
| 📍 Placement | Root and non-designated ports |
| ⛔ Conflict | Mutually exclusive with Root Guard |

---

## 💻 All Syntaxes Used

| # | Command | Mode | Description |
| --- | --- | --- | --- |
| 1 | `spanning-tree guard loop` | Interface config | Enables Loop Guard on a specific port |
| 2 | `spanning-tree loopguard default` | Global config | Enables Loop Guard on **all ports by default** |
| 3 | `spanning-tree guard none` | Interface config | Disables Loop Guard on a specific port (when enabled by default globally) |
| 4 | `spanning-tree guard root` | Interface config | Enables Root Guard on a port (disables Loop Guard on that port if present) |
| 5 | `show spanning-tree interface detail` | Privileged EXEC | Verifies Loop Guard status on an interface ("Loop guard is enabled on the port") |
| 6 | `show spanning-tree` | Privileged EXEC | Displays port status — shows **BKN** and **LOOP_Inc** when Loop Guard blocks a port |