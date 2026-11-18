# 🧠 CCNA Day 27: OSPF Part 2 - Metric, Neighbors, & Config

### 🎯 Main Topics

- **OSPF Metric (Cost):** How it's calculated and manipulated.
- **OSPF Neighbor States:** The step-by-step process of forming an adjacency.
- **OSPF Configuration:** Additional commands for interface-level setup and passive interfaces.

---

### 📏 OSPF Metric: Cost

- **Definition:** OSPF's metric is called **cost**. It's automatically calculated based on interface bandwidth.
- **Formula:** `Cost = Reference Bandwidth / Interface Bandwidth`
- **Default Reference Bandwidth:** **100 Mbps**
- **Key Rule:** Any calculated cost less than 1 is converted to **1**.
    - Ethernet (10 Mbps) = 10
    - FastEthernet (100 Mbps) = 1
    - Gigabit Ethernet (1000 Mbps) = 1
    - 10Gig Ethernet (10000 Mbps) = 1
- **Path Cost:** The total cost is the sum of the costs of all **outgoing (exit) interfaces** along the path.
    - **Loopback interfaces** always have a cost of **1**.
    - Example: R1 → R2 → R4 path with all GigE interfaces = 100 + 100 + 100 = **300** (with reference BW of 100,000 Mbps).

#### ⚙️ Modifying OSPF Cost

There are three ways to change the OSPF cost:

1. **Change the Reference Bandwidth (Recommended)**
    - Use a value *higher* than the fastest link in your network. Must be consistent across all routers.
    - **Syntax (OSPF Config Mode):**
        
        ```
        router ospf 1
        auto-cost reference-bandwidth <megabits-per-second>
        ```
        
2. **Manually Configure Interface Cost (Recommended)**
    - This value overrides the auto-calculated cost.
    - **Syntax (Interface Config Mode):**
        
        ```
        interface g0/0
        ip ospf cost <value>
        ```
        
3. **Change the Interface Bandwidth (Not Recommended)**
    - Changes the value used for calculations but **not the actual interface speed**.
    - **Syntax (Interface Config Mode):**
        
        ```
        interface g0/0
        bandwidth <kilobits-per-second>
        ```
        
    - **⚠️ Important:** `bandwidth` command is in **kilobits**, while `auto-cost reference-bandwidth` is in **megabits**. Use `?` to verify units.

#### ✅ Verification Command

- **Syntax:** `show ip ospf interface brief`
- **Purpose:** Provides a quick overview of OSPF-enabled interfaces, including cost, state, and neighbors.

---

### 🤝 OSPF Neighbor States

To become neighbors, routers must go through a 7-state process by exchanging OSPF messages.

- **Hello Messages:** Multicast to **224.0.0.5** every **10 seconds** (default on Ethernet).
- **Dead Timer:** **40 seconds**. If no Hello is received, the neighbor is removed.

#### The 7 Neighbor States (The Process)

1. **⬇️ Down:** No Hello packets have been received from the neighbor.
2. **📨 Init:** A Hello packet has been received, but the router's own Router ID is not included in the packet. This means the other router doesn't know about it yet.
3. **👋 2-Way:** Both routers have seen their own Router ID in the other's Hello packet. **This is the point where a basic neighbor relationship is formed.** (DR/BDR election happens here in some network types).
4. **🤝 Exstart:** The routers decide who will be the **Master** and who will be the **Slave** for the upcoming database exchange. The router with the **higher Router ID** becomes the Master.
5. **🔄 Exchange:** Routers send **Database Description (DBD)** packets to each other. These packets contain a summary of the LSAs in their Link-State Database (LSDB).
6. **⏳ Loading:** Routers send **Link-State Request (LSR)** packets to ask for any LSAs that were seen in the DBD but are missing from their own LSDB. The requested LSAs are sent in **Link-State Update (LSU)** packets.
7. **✅ Full:** The routers have successfully synchronized their LSDBs and have a **full OSPF adjacency**.

#### 📨 OSPF Message Types

- **Type 1: Hello** - Discovers and maintains neighbors.
- **Type 2: DBD** - Summarizes LSDB contents during Exstart/Exchange.
- **Type 3: LSR** - Requests specific LSAs during Loading.
- **Type 4: LSU** - Contains the full LSA information.
- **Type 5: LSAck** - Acknowledges receipt of an LSU.

#### ✅ Verification Command

- **Syntax:** `show ip ospf neighbor`
- **Purpose:** Displays the state of all OSPF neighbors (should show "FULL" state).

---

### ⚙️ Additional OSPF Configuration

#### 1. Activating OSPF Directly on an Interface

Instead of using the `network` command in OSPF config mode, you can enable OSPF per-interface.

- **Syntax (Interface Config Mode):**
    
    ```
    interface g0/0
    ip ospf <process-id> area <area-id>
    ```
    
- **Verification:** `show ip protocols` will now list the interfaces under "Routing on Interfaces Configured Explicitly".

#### 2. Passive Interfaces (Alternative Method)

You can set all interfaces as passive by default, then selectively make them active. This is useful when most interfaces are passive.

- **Syntax (OSPF Config Mode):**
    
    ```
    router ospf 1
    passive-interface default
    no passive-interface g0/0  // 'no' to make an interface active
    ```
    

---

### 📝 Quiz from the Video

**Q1: Put the OSPF neighbor states in the correct order.**

> **Answer:** Down → Init → 2-way → Exstart → Exchange → Loading → Full.
> 

**Q2: Which statement about OSPF's default cost is correct?**
a) All interfaces have the same cost.
b) Ethernet and FastEthernet interfaces have the same cost.
c) FastEthernet, Gigabit Ethernet, and 10Gig Ethernet interfaces have the same cost.
d) Ethernet, FastEthernet, Gigabit Ethernet, and 10Gig Ethernet interfaces have the same cost.

> **Answer: C** — FastEthernet, Gigabit Ethernet, and 10Gig Ethernet all have a default cost of 1 because any value less than 1 is converted to 1 (with default reference bandwidth of 100 Mbps).
> 

**Q3: In which OSPF neighbor state are the Master and Slave roles decided?**
A) Exstart
B) 2-way
C) Exchange
D) Loading

> **Answer: A** — Exstart. The router with the higher Router ID becomes Master and initiates the DBD exchange in the next state (Exchange). In the 2-way state, DR/BDR elections happen (in some network types), not Master/Slave.
> 

**Q4: Which of these commands can be used to make a FastEthernet interface have an OSPF cost of 100?**
A) auto-cost reference-bandwidth 100
B) auto-cost reference-bandwidth 1000
C) auto-cost reference-bandwidth 10,000
D) auto-cost reference-bandwidth 100,000

> **Answer: C** — `auto-cost reference-bandwidth 10,000`. FastEthernet bandwidth = 100 Mbps. 10,000 / 100 = 100.
> 

**Q5: What are the default OSPF Hello / Dead timers on an Ethernet connection?**
A) Hello 2s, Dead 20s
B) Hello 10s, Dead 40s
C) Hello 30s, Dead 120s
D) Hello 60s, Dead 180s

> **Answer: B** — Hello 10s, Dead 40s on Ethernet. (Note: Some other connection types use 30s/120s, covered in Day 28.)
> 

---

### 🎁 Bonus Question (Boson ExSim)

**Scenario:** `auto-cost reference-bandwidth 1000` issued on every router. What is the cost of the route from RouterA to RouterC?

- Gigabit Ethernet interfaces = cost **1** (1000/1000)
- FastEthernet interfaces = cost **10** (1000/100)
- **Best path:** RouterA → RouterB → RouterE → RouterC = 1 + 1 + 1 = **3**
- **Answer: B (3)** — Avoids the FastEthernet links which cost 10 each.

---

### 📋 Syntax Summary Table

| Command | Mode | What It Does |
| --- | --- | --- |
| `auto-cost reference-bandwidth <Mbps>` | OSPF Config (`router ospf 1`) | Changes the reference bandwidth used for OSPF cost calculation. Entered in **megabits per second**. Should be consistent across all routers. |
| `ip ospf cost <value>` | Interface Config (`interface g0/0`) | Manually sets the OSPF cost of the interface. Overrides the auto-calculated cost. |
| `bandwidth <Kbps>` | Interface Config (`interface g0/0`) | Changes the interface bandwidth value used for OSPF/EIGRP calculations. Entered in **kilobits per second**. Does **not** change actual interface speed. **Not recommended** for altering OSPF cost. |
| `ip ospf <process-id> area <area-id>` | Interface Config (`interface g0/0`) | Enables OSPF directly on the interface without using the `network` command in OSPF config mode. |
| `passive-interface default` | OSPF Config (`router ospf 1`) | Sets **all** interfaces as passive by default. |
| `no passive-interface <interface>` | OSPF Config (`router ospf 1`) | Makes a specific interface active (not passive) when `passive-interface default` is configured. |
| `show ip ospf interface <interface>` | Privileged EXEC (`#`) | Displays detailed OSPF info for an interface: cost, Hello/Dead timers, neighbor count, DR/BDR status. |
| `show ip ospf interface brief` | Privileged EXEC (`#`) | Quick overview table of all OSPF-enabled interfaces: cost, state, neighbors. |
| `show ip ospf neighbor` | Privileged EXEC (`#`) | Displays OSPF neighbor states, Dead timer countdown, and DR/BDR roles. |
| `show ip protocols` | Privileged EXEC (`#`) | Shows routing protocol configuration; verify OSPF is enabled and which interfaces are active (via `network` or explicit interface config). |

---

### 🔑 Key Takeaways

- Default reference bandwidth = **100 Mbps**; anything ≥100 Mbps gets cost **1** by default.
- Best practice: **increase reference bandwidth** to accommodate future faster links + use `ip ospf cost` for fine-tuning.
- The **2-Way state** is where basic neighbor relationship is formed; **Full state** = synchronized LSDBs.
- Master/Slave decided in **Exstart** state (higher Router ID wins).
- Hello = **10s**, Dead = **40s** on Ethernet.
- OSPF messages multicast to **224.0.0.5**; IP protocol number **89**.