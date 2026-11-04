# 🛡️ Root Guard: Protecting Your STP Root Bridge

## 🎯 What is Root Guard?

- **Definition:** An STP toolkit feature that guards the root bridge of the LAN. It ensures that another switch with a lower bridge ID cannot take over the role.
- **Mechanism:** It prevents a port from becoming a root port. It does this by disabling the port if it receives a **superior BPDU**.
- **Use Case:** Typically used when connecting your switches to a network you don’t have direct control over (e.g., a service provider connecting to a customer).
- **Video Context:** This is part of the STP Toolkit, following PortFast, BPDU Guard, and BPDU Filter. The next topic will be Loop Guard.

---

## 🏗️ Why is Root Bridge Selection Important?

Before implementing Root Guard, you must understand why you want a specific switch to remain the root bridge.

- **Optimal Traffic Flow:**
    - The root bridge should be placed to minimize latency and congestion.
    - *Example:* If hosts primarily communicate with servers outside the LAN (via a gateway), the switch closest to the gateway (e.g., SW1) should be the root.
    - *Risk:* If a less optimal switch becomes root, traffic takes a longer, less direct path and may cause congestion on specific links.
- **Stability and Reliability:**
    - The root bridge should be a stable, reliable switch (e.g., newer hardware) to prevent unnecessary topology changes.

> **💡 Note:** Within your own LAN, you can manually control the root bridge by setting its priority to **0**. However, another switch with a lower MAC address can still take over if priority is equal.
> 

---

## 🚨 The Problem: Uncontrolled Networks

When two networks managed by different organizations connect, they might disagree on who the root bridge should be.

- **Scenario:**
    - **Service Provider LAN:** SW1 is manually set as the root bridge (Priority 0).
    - **Customer LAN:** SW6 is manually set as the root bridge (Priority 0).
    - **Conflict:** SW6 has a lower MAC address than SW1.
    - **Result:** SW1, SW2, and SW3 accept SW6 as the new root bridge.
- **Consequence:**
    - The service provider's topology is disrupted.
    - Links between the provider’s switches (SW1-SW3, SW2-SW3) may be disabled, forcing traffic to detour through the customer’s network.
    - This is inefficient and causes latency/congestion.

---

## ⚙️ How Root Guard Works

Root Guard is the safeguard against this scenario.

- **Configuration:**
    - Applied to the **interface** connected to the external (untrusted) switch.
    - **Command:** `spanning-tree guard root`
    - *Note:* Unlike BPDU Guard/Filter, there is **no global config command** to enable this by default. It must be applied per-interface.
- **The Trigger:**
    - The Root Guard-enabled port receives a **Superior BPDU** (a BPDU claiming a better root bridge ID than the current root).
- **The Action:**
    - The port enters a **Broken** / **Root Inconsistent** state.
    - **Broken:** The port is effectively disabled; it cannot forward or receive data frames.
- **The Result:**
    - The superior BPDUs are effectively blocked from influencing the rest of the network.
    - SW1 remains the root bridge for the service provider’s LAN.

---

## 🔄 Recovery and Resolution

A port disabled by Root Guard does not require manual intervention (like `shutdown` / `no shutdown`) to recover.

1. **Fix the Issue:** The external network (customer) must stop sending superior BPDUs.
    - *Example:* The customer increases their switch's (SW6) priority (e.g., to 4096) so it is no longer superior.
2. **Automatic Recovery:**
    - Once the superior BPDUs age out (default Max Age is **20 seconds**), the port automatically transitions back to a normal state.
    - The network re-converges with the intended root bridge (SW1).

---

## 📝 Key Takeaways & Syntax

### ✅ Syntax

- **Enable Root Guard:**
    
    ```
    interface GigabitEthernet0/2
     spanning-tree guard root
    ```
    

### ⚠️ Important Points

- **Where to Configure:**
    - **Do:** Configure on the **Service Provider** side (ports facing the customer) to protect the root.
    - **Don't:** Configure on the **Customer** side (ports facing the provider). If you do, the customer's ports will just be disabled, blocking connectivity entirely.
- **State Verification:**
    - Use `show spanning-tree` to verify the port status.
    - Look for **BKN** (Broken) and **ROOT_Inc** (Root Inconsistent) in the output.
- **No Global Config:** Unlike `spanning-tree portfast default` or `spanning-tree portfast bpduguard default`, Root Guard must be enabled manually on specific interfaces. It is not a feature you want to enable on all ports by default.