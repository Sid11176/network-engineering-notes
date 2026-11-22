# 📡 APNIC and CCNA: BGP Configuration (eBGP Single-Homed)

## 🎯 Lesson Overview

Configure **external BGP (eBGP)** for single-homed branch connectivity to an ISP. This lab covers BGP fundamentals, neighbor configuration, network advertisement, and discard routes.

---

## 🌍 BGP Fundamentals

### What is BGP?

- **BGP** = Border Gateway Protocol — the only **EGP** (Exterior Gateway Protocol) in use today
- Unlike IGPs (RIP, EIGRP, OSPF) used **within** an organization, BGP routes **between** organizations
- **Path Vector Protocol** — uses **AS path** instead of hop count
- Relies on **incremental updates**
- Supports **CIDR** and many policy options
- Runs over **TCP port 179**

### Autonomous Systems (AS)

| Concept | Details |
| --- | --- |
| **AS** | A network under single administrative control |
| **ASN Size** | 32-bit (historically 16-bit: 0–65535) |
| **Public Range** | ~1 to 64,000 |
| **Private Range** | 64512 to 65534 |
| **New 32-bit Range** | 65552 and above |

---

## 🏠 iBGP vs 🌍 eBGP

| Feature | iBGP | eBGP |
| --- | --- | --- |
| **Purpose** | Carry prefixes inside an AS | Exchange prefixes between ASes |
| **Direct Connection?** | No | Yes — must be directly connected |
| **Full Mesh Required?** | Yes | No |
| **Best Practice** | Peer between loopbacks | Peer on direct link |

### Key Rules

- **eBGP**: Different AS numbers → automatically eBGP
- **iBGP**: Same AS number → automatically iBGP
- iBGP does **not** re-advertise routes learned from another iBGP peer (prevents loops)
- Always use `update-source loopback 0` for iBGP loopback peering

---

## 🔧 Lab Configuration Steps

### Scenario

- **Our AS**: 65000
- **ISP AS**: 65501
- **Goal**: Establish eBGP peering between R1 and ISP router, advertise internal networks

### Step 1: Configure BGP on R1

```
R1(config)# router bgp 65000
R1(config-router)# neighbor 10.1.1.1 remote-as 65501
```

⚠️ **Key Point**: Unlike OSPF/EIGRP, BGP requires **manual neighbor specification** — no dynamic discovery.

### Step 2: Advertise Networks

```
R1(config-router)# network 1.1.1.1 mask 255.255.255.255
R1(config-router)# network 2.2.2.2 mask 255.255.255.255
R1(config-router)# network 3.3.3.3 mask 255.255.255.255
```

⚠️ **Critical Difference**: BGP `network` command requires:

- **Exact network and mask** (not a wildcard)
- A **matching route must exist** in the routing table

### Step 3: Create Discard Route for Summary

Problem: No 10.0.0.0/16 route exists in routing table to advertise.

Solution: Create a static discard route:

```
R1(config)# ip route 10.0.0.0 255.255.0.0 null0
R1(config-router)# network 10.0.0.0 mask 255.255.0.0
```

### Why Use null0 (Discard Route)?

- **Advertises aggregate** without depending on more-specific routes
- **Silently discards** traffic for unallocated space (e.g., 10.1.0.0/24)
- More-specific routes (/30) still forward traffic correctly
- Prevents packets from rattling around until TTL expires
- **Security benefit**: Stops unwanted traffic at the edge

---

## 📥 Injecting Prefixes into BGP

| Method | Requirement | Notes |
| --- | --- | --- |
| **`network` statement** | Matching route in RIB | Instructor's preferred method |
| **`redistribute static`** | Static route exists | Use with route map to filter |

### Aggregates: Best Practice

✅ **Preferred**: Static route to null0 + matching `network` statement

- Aggregate never disappears
- Stable and predictable

❌ **Avoid**: `aggregate-address` command

- Aggregate disappears if more-specific routes disappear
- Can cause instability and make you "disappear" from the internet

---

## ⌨️ Syntax Enumeration

| Syntax | Mode | Description |
| --- | --- | --- |
| `router bgp (AS_NUMBER)` | Global Config | Starts BGP process and enters BGP configuration mode |
| `neighbor (IP) remote-as (AS_NUMBER)` | BGP Config | Defines a BGP neighbor and its remote AS |
| `neighbor (IP) update-source loopback 0` | BGP Config | Sources BGP packets from Loopback 0 (mandatory for iBGP loopback peering) |
| `network (NETWORK) mask (MASK)` | BGP Config | Injects prefix into BGP if matching route exists in RIB |
| `redistribute static` | BGP Config | Redistributes static routes into BGP (use with route map) |
| `aggregate-address (NETWORK) (MASK)` | BGP Config | Creates aggregate route (⚠️ risky — can disappear) |
| `ip route (NETWORK) (MASK) null0` | Global Config | Creates discard route for aggregate advertisement |
| `show ip bgp summary` | Privileged EXEC | Shows BGP neighbor summary, uptime, prefix counts |
| `show ip route` | Privileged EXEC | Displays routing table |

### 32-Bit ASN DOT Notation

- `2.4` = `131076`
- Calculation: `2 × 65536 + 4 = 131076`
- ⚠️ DOT notation can break BGP policies — avoid unnecessarily

---

## 📊 show ip bgp summary — What You See

- List of neighbors
- BGP version number
- Local AS number
- Input/output queue counts
- Peering uptime
- Prefixes received from each peer
- BGP table information

---

## ✅ Key Takeaways

1. **BGP** is the only EGP in use — routes between organizations
2. **eBGP** requires direct connection; **iBGP** does not
3. BGP `network` command needs **exact mask** + matching route in RIB
4. Use **null0 discard routes** for stable aggregate advertisement
5. **Avoid `aggregate-address`** — aggregates can disappear
6. BGP uses **TCP port 179**
7. Private ASNs: **64512–65534**

---

## ❓ Quiz

**1. What is the main connection requirement difference between iBGP and eBGP peers?**

**Answer:** eBGP peers must be directly connected; iBGP peers do not need to be directly connected.

---

**2. Why should iBGP sessions be established between loopback interfaces?**

**Answer:** Loopbacks are not tied to one physical link, so the iBGP session stays up if a physical link fails, as long as an alternate path to the loopback exists.

---

**3. What is the iBGP full-mesh rule, and why does it exist?**

**Answer:** Every iBGP router must peer with every other iBGP router in the AS. It exists because iBGP does not re-advertise routes learned from one iBGP peer to another, preventing loops and duplicate updates.

---

**4. What are the two main ways to inject prefixes into BGP?**

**Answer:** Use the `network` statement or `redistribute`. `redistribute` should be used with a route map to avoid injecting unwanted routes.

---

**5. Why does the instructor avoid the `aggregate-address` command?**

**Answer:** The aggregate can disappear if the more-specific routes it depends on are withdrawn, causing instability. A static route to null0 plus a matching `network` statement is safer.

---

**6. What does `2.4` mean in 32-bit ASN DOT notation?**

**Answer:** It equals ASN `131076`, calculated as `2 × 65536 + 4`.

---

**7. What port does BGP use?**

**Answer:** TCP port **179**.