# 🎯 Day 29: First Hop Redundancy Protocols (FHRPs)

### 📝 The Purpose of FHRPs

- **The Problem:** If a network has only one default gateway router and it fails, all hosts lose connectivity to other networks. Hosts are manually configured with a single default gateway IP and won't automatically switch to a backup.
- **The Solution:** An FHRP provides a **redundant default gateway** by allowing two or more routers to share a **Virtual IP (VIP)** and a **Virtual MAC (vMAC)** address.
    - End hosts are configured with this VIP as their default gateway.
    - An **Active** router handles traffic, while a **Standby** router waits to take over if the active router fails.

### ⚙️ How FHRPs Work: The Process

1. **Negotiation:** Routers send multicast **Hello** messages to elect an Active and a Standby router.
2. **ARP Resolution:** When a host ARPs for its default gateway (the VIP), only the **Active router** replies, providing the **Virtual MAC** address.
3. **Traffic Flow:** The host sends all traffic for other networks to the vMAC address, which is processed by the Active router.
4. **Failover:** If the Active router fails, the Standby router stops receiving Hellos and takes over as the new Active router.
5. **Network Update:** The new Active router sends a **Gratuitous ARP (GARP)** to update the MAC address tables of all switches, directing traffic for the vMAC to its own physical port.
6. **Preemption:** By default, if the original router comes back online, it does not take back its Active role (non-preemptive). This can be changed with the `preempt` command.

---

### 🏆 The 3 Main FHRPs

#### 1. HSRP (Hot Standby Router Protocol)

- **Type:** **Cisco Proprietary** 🤫
- **Roles:** **Active** and **Standby** routers.
- **Key Feature:** Can load balance between different subnets/VLANs by using a different active router for each.

#### 2. VRRP (Virtual Router Redundancy Protocol)

- **Type:** **Open Standard** 🌐
- **Roles:** **Master** and **Backup** routers.
- **Key Feature:** Functionally very similar to HSRP.

#### 3. GLBP (Gateway Load Balancing Protocol)

- **Type:** **Cisco Proprietary** 🤫
- **Roles:** **AVG (Active Virtual Gateway)** and up to 4 **AVFs (Active Virtual Forwarders)** .
- **Key Feature:** **Load balances traffic within a single subnet**. The AVG assigns different vMACs to each AVF, and different hosts use different AVFs as their gateway.

### 📊 FHRP Comparison Chart

| Feature | HSRP | VRRP | GLBP |
| --- | --- | --- | --- |
| **Standard** | Cisco Proprietary | Open Standard | Cisco Proprietary |
| **Router Roles** | Active, Standby | Master, Backup | AVG, AVF |
| **Load Balancing** | Per Subnet/VLAN | Per Subnet/VLAN | **Per Host (in a single subnet)** |
| **Multicast IPv4** | **v1:** `224.0.0.2` 
**v2:** `224.0.0.102` | `224.0.0.18` | `224.0.0.102` |
| **Virtual MAC** | **v1:** `0000.0c07.acXX` 
**v2:** `0000.0c9f.fXXX` | `0000.5e00.01XX` | `0007.b400.XXYY` |
| `*XX` = HSRP/VRRP Group Number, `YY` = GLBP AVF Number* |  |  |  |

---

### 💻 HSRP Configuration Syntax

These commands are executed in **Interface Configuration Mode**.

| Command | Description |
| --- | --- |
| `standby version 2` | Sets the HSRP version to 2 for the interface (must match on all routers in the group). |
| `standby [group-number] ip [virtual-ip]` | Configures the virtual IP address that hosts will use as their default gateway. |
| `standby [group-number] priority [value]` | Sets the priority (0-255, default 100). Higher priority wins the Active role election. |
| `standby [group-number] preempt` | Enables preemption, allowing a router to take back the Active role if it recovers, provided it has the highest priority. |

**Example Configuration:**

```
! On R1 (intended Active Router)
interface GigabitEthernet0/0
 standby version 2
 standby 1 ip 172.16.0.254
 standby 1 priority 200
 standby 1 preempt
!
! On R2 (intended Standby Router)
interface GigabitEthernet0/0
 standby version 2
 standby 1 ip 172.16.0.254
!
```

**Verification Command:**

- `show standby`: Executed in **Privileged EXEC Mode**. Displays the HSRP status, including the Active/Standby roles, virtual IP/MAC, and timers.

---

### 🧠 Quiz Time!

**1. Which of the following is an HSRP version 1 virtual MAC address?**
A. `0000.0c9f.f001`
B. `0007.b400.0101`
C. `0000.5e00.0101`
D. `0000.0c07.acab`

**2. Which of the following is a VRRP virtual MAC address?**
A. `0000.5e00.010a`
B. `0007.b400.0101`
C. `0000.0c9f.f001`
D. `0000.0c07.acab`

**3. Which of the following are valid VRRP router roles? (Select two)**
A. Active
B. Backup
C. Active Virtual Gateway
D. Master
E. Standby
F. Active Virtual Forwarder

**4. When the HSRP standby router becomes the new active router, what kind of messages does it send?**
A. HSRP hello messages
B. Gratuitous ARP messages
C. ARP request messages
D. ARP reply messages

**5. Which of the following statements accurately describes HSRP?**
A. It allows you to configure a different default gateway address on each end host.
B. It allows multiple routers to load-balance traffic for hosts in a single subnet.
C. It provides a redundant default gateway address for hosts in a subnet.
D. Each router selects a unique virtual IP and MAC address.

**Answers:**

1. **D.** The HSRPv1 format is `0000.0c07.ac` + group number (in hex).
2. **A.** The VRRP format is `0000.5e00.01` + group number (in hex).
3. **B, D.** VRRP uses Master and Backup routers. Active/Standby is HSRP; AVG/AVF is GLBP.
4. **B.** The new active router sends Gratuitous ARP to update the switches' MAC address tables.
5. **C.** HSRP provides a redundant default gateway. It cannot load balance within a single subnet (that's GLBP).

---

### 📝 Boson ExSim Practice Question

**Which of the following statements are true regarding HSRP? (Select two choices.)**

- **A.** One router is elected as the master router, and all other routers are placed in the backup state.
- **B.** In an HSRP group, only one virtual IP address is used, but multiple virtual MAC addresses are used.
- **C.** One router is elected as the active router and another router is elected as the standby router.
- **D.** In an HSRP group, only one virtual IP address and one virtual MAC address is used.
- **E.** All routers in an HSRP group can participate by forwarding a portion of the traffic.

### ✅ Correct Answers: **C and D**

**Explanation (from Boson):**

- **A is incorrect** – "Master" and "backup" are VRRP terms, not HSRP.
- **B is incorrect** – In a single HSRP group, only **one** virtual MAC address is used.
- **C is correct** – HSRP elects one **active** and one **standby** router.
- **D is correct** – An HSRP group uses exactly **one virtual IP** and **one virtual MAC**.
- **E is incorrect** – That describes **GLBP**, not HSRP.