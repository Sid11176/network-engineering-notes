# 🌐 CCNA Day 53 — WAN Architectures, VPNs & GRE Lab

> 📚 **Course:** Jeremy's IT Lab — CCNA 200-301
🎯 **Exam topics:** 1.2.d (WAN) · 5.5 (Remote-access & site-to-site VPNs)
🧭 **Focus:** Concepts only (no config) for the lesson. The GRE lab adds hands-on config.
> 

---

# 📖 PART 1 — Lesson: WAN & VPNs

## 🏢 1. WAN Basics

- **WAN** = network spanning a large area, connecting geographically separated LANs.
- In enterprise terms, "WAN" usually means the **private links between company sites**, not the public Internet.
- Availability differs by country, so legacy tech may still be in use somewhere.

### ⭐ Hub-and-Spoke Topology

- **Hub** = central site (often a data center). **Spokes** = remote offices.
- ✅ Centralized firewall, security policy, and traffic control.
- ⚠️ Spoke-to-spoke traffic must pass through the hub (vs. direct in a full mesh).

### 🛰️ Provider Role

- Sites connect to a **service provider**, not directly to each other.
- Traditional serial WAN links use Layer 2 encapsulation **HDLC** or **PPP** (not Ethernet).
- **Ethernet WAN** (often over fiber) is increasingly common: high speed over long distances.

---

## 🔌 2. Leased Lines

- **Dedicated, private, non-shared** physical link between two sites. Not on the public Internet.
- Traditional: serial links with **PPP / HDLC**.
- ❌ Downsides vs Ethernet WAN: higher cost, longer install time, lower speed.

| Region | Standard | Examples |
| --- | --- | --- |
| 🇺🇸 North America | T-carrier | T1, T2, T3 |
| 🇪🇺 Europe / others | E-carrier | E1, E2, E3 |

> ⭐ **Memorize: T1 = 1.544 Mbps.** Be familiar with T/E series, but don't memorize every speed.
> 

---

## 🏷️ 3. MPLS VPNs

**MPLS (Multiprotocol Label Switching)**: connects sites over a provider's **shared** infrastructure while keeping each customer's traffic **logically separated**.

- **Labels** separate customers' traffic.
- Provider routers forward by **label**, not by destination IP, inside the MPLS network.
- Label sits **between L2 and L3 headers** → called **Layer 2.5 protocol**.
- Customer access can be fiber Ethernet, 4G/5G, CATV, or serial leased line.

### 🧩 Router Roles

| Router | Name | Role |
| --- | --- | --- |
| **CE** | Customer Edge | Customer-side router. **Does NOT need to run MPLS** ⭐ |
| **PE** | Provider Edge | Connects to CE, **adds MPLS labels** |
| **P** | Provider (core) | Internal provider router, **no direct link to customers** |

### ⚖️ Layer 3 vs Layer 2 MPLS VPN

| Feature | 🅻3 MPLS VPN | 🅻2 MPLS VPN |
| --- | --- | --- |
| CE–PE relationship | CE **peers with PE** (OSPF, other protocol, or static with PE as next hop) | CE does **not** peer with PE |
| Routing | Provider participates in L3 routing | CE routers can peer **directly with each other** (e.g. OSPF) |
| Customer view | Provider routes your traffic | Provider looks like **one big switch** |
| WAN interfaces | Not one shared subnet | Both CE WAN interfaces can be in the **same subnet** |

> 🧠 **Remember:** L3 = CE↔PE routing. L2 = provider acts like a switch, CE↔CE adjacency.
> 

---

## 🌍 4. Enterprise Internet Connectivity

- **DSL** 📞: Internet over **telephone lines**; needs a **modem** (can be separate or built into router).
- **Cable Internet** 📺: over **CATV/coaxial** lines; needs a **cable modem**.
- **Fiber Ethernet** 💡: increasingly popular, high speed over long distance.
- Private WAN (leased line/MPLS) can also provide a path to the provider's Internet.

### 🔁 Internet Redundancy Terms ⭐

| Term | Connections | Meaning |
| --- | --- | --- |
| **Single-homed** | 1 link to 1 ISP | No redundancy |
| **Dual-homed** | 2 links to the **same** ISP | Link redundancy, still one ISP |
| **Multihomed** | 1 link to **each of 2** ISPs | ISP redundancy |
| **Dual multihomed** | 2 links to **each of 2** ISPs | Highest redundancy |

> ⚠️ **Exam trap:** Dual-homed = 2 links, **one** ISP. Multihomed = **multiple** ISPs.
> 

---

## 🔐 5. Internet VPNs

The Internet is shared and not private/secure by default, so a **VPN** creates protected communication over it.

### 🏬↔️🏬 5.1 Site-to-Site VPN (IPsec)

- Connects **two sites** over the Internet. Typically **IPsec**.
- Tunnel endpoints are usually **routers or firewalls**. End hosts need **no** VPN software.
- One tunnel protects traffic for **many devices** at both sites.

**Packet flow:**

1. Host sends packet to its default gateway.
2. VPN endpoint **encrypts** the original packet.
3. Adds IPsec encapsulation + **new outer IP header**.
4. Packet crosses the Internet.
5. Remote endpoint **decapsulates and decrypts**.
6. Original packet is forwarded to destination host.

⚠️ **Standard IPsec limitation:** supports **unicast only**, no broadcast/multicast. So routing protocols like **OSPF (multicast)** can't run over it directly. Fixes: **GRE over IPsec** and **DMVPN**.

### 📦 5.2 GRE over IPsec

- **GRE** = tunneling protocol that can encapsulate many L3 protocols, **including multicast/broadcast**.
- ❌ GRE alone **does NOT encrypt**, so it is not secure.
- **GRE over IPsec** = GRE flexibility 🤝 IPsec encryption.

**Encapsulation order:**

1. Original IP packet → 2. Add **GRE header + new IP header** → 3. **Encrypt** → 4. Add **IPsec header + another new IP header**

### 🕸️ 5.3 DMVPN (Dynamic Multipoint VPN)

- **Cisco** solution: dynamically builds a **full mesh** of IPsec tunnels without configuring every tunnel manually.
- How: each spoke has a tunnel to the **hub** → hub shares info → spokes build **direct spoke-to-spoke tunnels**.
- ✅ Simple initial config (only tunnel to hub), efficient direct paths.
- 💡 A company may still prefer hub-routed traffic for centralized firewall inspection.

### 📱 5.4 Remote-Access VPN (TLS)

- Lets an **individual end device** (laptop/phone) securely reach company resources over the Internet.
- Typically **TLS** (same tech securing **HTTPS**). TLS = renamed successor of **SSL**.
- Needs **VPN client software** on the device (e.g. **Cisco AnyConnect**).
- Client builds a tunnel to a company router/firewall acting as **TLS server**.

### ⚔️ Site-to-Site vs Remote-Access ⭐

| Feature | Site-to-Site | Remote-Access |
| --- | --- | --- |
| Protocol | **IPsec** | **TLS** |
| Connects | Two sites | One device → company network |
| Endpoints | Routers / firewalls | VPN client ↔ VPN server |
| Who benefits | Many devices at both sites | The device running the client |
| Use | Persistent site connectivity | On-demand user access |

---

## 🧠 6. High-Priority Review

- 🌐 **WAN** connects separated LANs · **Hub-and-spoke** centralizes control
- 🔌 **Leased line** = dedicated, serial **PPP/HDLC** · **T1 = 1.544 Mbps**
- 🏷️ **MPLS** = label forwarding in provider network · **CE** doesn't run MPLS · **PE** adds labels · **P** = core
- 🅻3 MPLS VPN = **CE–PE routing** · 🅻2 MPLS VPN = **provider acts like a switch**
- 📞 **DSL** = telephone lines · 📺 **Cable** = CATV lines
- 🔁 Single / Dual-homed / Multihomed / Dual multihomed → know **links vs ISPs**
- 🏬 **Site-to-site = IPsec** · 📱 **Remote-access = TLS**
- 📦 **GRE** = multicast/broadcast capable, **no encryption** · **GRE over IPsec** = both
- 🕸️ **DMVPN** = dynamic spoke-to-spoke IPsec full mesh

---

# 🧪 PART 2 — Lab: GRE Tunnels

## 🎯 Lab Objectives

- Build a GRE tunnel between **R1** and **R2** across a service provider network.
- Understand GRE encapsulation.
- Verify and troubleshoot tunnel status.
- Run **OSPF over the tunnel** to exchange LAN routes.
- Get **PC1 ↔ PC2** communicating through the tunnel.

## 🧠 GRE Fundamentals

- Creates a **virtual point-to-point link** between two routers over an existing network.
- R1 and R2 look **directly connected**, but traffic physically crosses the provider network.
- Original packet gets a **GRE header + outer IP header** (outer IP lets it cross the provider).
- 🚫 **No encryption.**

## 🗺️ Topology & Addressing

| Interface | IP address | Purpose |
| --- | --- | --- |
| R1 G0/0/0 | 100.0.0.2 | R1 WAN |
| R2 G0/0/0 | 200.0.0.2 | R2 WAN |
| R1 Tunnel0 | 192.168.1.1/30 | GRE endpoint |
| R2 Tunnel0 | 192.168.1.2/30 | GRE endpoint |
| R1 LAN | 10.0.1.0/24 | PC1 network |
| R2 LAN | 10.0.2.0/24 | PC2 network |
| R1 next hop | 100.0.0.1 | Provider gateway |
| R2 next hop | 200.0.0.1 | Provider gateway |

### 🔑 Key Distinctions

- `tunnel source` → **local** physical/WAN interface used for the tunnel
- `tunnel destination` → **remote router's WAN IP**
- `ip address` (under tunnel) → address of the **virtual tunnel interface**
- Tunnel destination must be **reachable via the underlying network**

## ⚙️ Step 1 — GRE Configuration

**R1**

```
enable
configure terminal
interface tunnel 0
 tunnel source g0/0/0
 tunnel destination 200.0.0.2
 ip address 192.168.1.1 255.255.255.252
exit
ip route 0.0.0.0 0.0.0.0 100.0.0.1
```

**R2**

```
enable
configure terminal
interface tunnel 0
 tunnel source g0/0/0
 tunnel destination 100.0.0.2
 ip address 192.168.1.2 255.255.255.252
exit
ip route 0.0.0.0 0.0.0.0 200.0.0.1
```

> 💡 **Why default routes?** Each router must be able to reach the other's WAN IP before the tunnel can come up (R1 → 200.0.0.2, R2 → 100.0.0.2).
> 

> ⚠️ **Key troubleshooting point:** A configured tunnel is **not** automatically operational. If the tunnel destination is unreachable, Tunnel0 stays **down**.
> 

## 🔍 Step 2 — Verify & Troubleshoot

1. `do show ip interface brief` → Tunnel0 may show **up/down** or down if the remote endpoint is unreachable.
2. `do show ip route` → check for a route to the remote WAN IP.
3. Add the missing default route if needed.
4. Test: R1 `do ping 192.168.1.2` · R2 `do ping 192.168.1.1`

> 📝 **Packet Tracer note:** first 1–2 pings may fail while ARP resolves. Later pings succeed.
> 

✅ **Expected:** Tunnel0 up · connected route for `192.168.1.0/30` · endpoints ping each other.

## 📦 Step 3 — Encapsulation (Simulation Mode)

When R1 pings R2's tunnel IP:

| Header / Packet | Source | Destination |
| --- | --- | --- |
| Original IP packet | 192.168.1.1 | 192.168.1.2 |
| Outer IP header | 100.0.0.2 | 200.0.0.2 |
1. R1 creates the original packet to R2's tunnel IP.
2. GRE encapsulates it.
3. Outer IP header added using **WAN addresses**.
4. Provider forwards based on the **outer IP**.
5. R2 decapsulates and recovers the original packet.

> 🧠 **Provider sees the outer packet only.** The original is carried inside the tunnel.
> 

Packet layout: `Outer IP Header (100.0.0.2 → 200.0.0.2)` → `GRE Header` → `Original IP Packet (192.168.1.1 → 192.168.1.2)`

## 🔄 Step 4 — OSPF over GRE

GRE makes R1 and R2 **virtually directly connected**, so they can form an **OSPF neighbor** relationship over Tunnel0 and exchange LAN routes. Before OSPF, **PC1 cannot reach PC2** (no remote LAN routes).

**R1**

```
configure terminal
router ospf 1
 network 192.168.1.1 0.0.0.0 area 0
 network 10.0.1.1 0.0.0.0 area 0
 passive-interface g0/0
```

**R2**

```
configure terminal
router ospf 1
 network 192.168.1.2 0.0.0.0 area 0
 network 10.0.2.1 0.0.0.0 area 0
 passive-interface g0/0
```

| Command | Purpose |
| --- | --- |
| `router ospf 1` | Starts OSPF process 1 |
| `network 192.168.1.x 0.0.0.0 area 0` | Enables OSPF on **Tunnel0** |
| `network 10.0.x.1 0.0.0.0 area 0` | Advertises the **LAN** network |
| `passive-interface g0/0` | No neighbors on LAN, but network is still advertised |

**Key points:**

- OSPF on Tunnel0 → neighbors form. OSPF on LAN → LAN advertised.
- LAN interfaces are **passive** because no OSPF routers sit there.
- Process ID `1` is **locally significant** (doesn't need to match).
- Both routers use **Area 0**.

**Learned routes (`show ip route`):**

| Router | Remote LAN | Via |
| --- | --- | --- |
| R1 | 10.0.2.0/24 | Tunnel0 |
| R2 | 10.0.1.0/24 | Tunnel0 |

🏁 **Final test:** from PC1 → `ping 10.0.2.100` ✅

---

# ⌨️ PART 3 — Syntax & Command Reference

> 📌 The lesson (Part 1) had **no CLI commands**. All commands below come from the **GRE lab**.
> 

| Syntax | Description | Execution mode |
| --- | --- | --- |
| `enable` | Enters privileged EXEC mode | User EXEC (`>`) |
| `configure terminal` / `conf t` | Enters global config mode | Privileged EXEC (`#`) |
| `interface tunnel 0` | Creates/enters Tunnel0 | Global configuration |
| `tunnel source g0/0/0` | Sets local physical tunnel source interface | Interface configuration |
| `tunnel destination 200.0.0.2` | Sets remote WAN endpoint | Interface configuration |
| `ip address 192.168.1.1 255.255.255.252` | Assigns IP to Tunnel0 | Interface configuration |
| `exit` | Exits the current config mode | Configuration modes |
| `ip route 0.0.0.0 0.0.0.0 <next-hop>` | Default static route | Global configuration |
| `router ospf 1` | Enters OSPF router config mode | Global configuration |
| `network <IP> <wildcard> area <area>` | Enables OSPF on matching interfaces and advertises their networks | OSPF router configuration |
| `passive-interface g0/0` | Stops OSPF neighbor formation on that interface | OSPF router configuration |
| `show ip interface brief` | Shows interface status and IPs | Privileged EXEC |
| `show ip route` | Shows the routing table | Privileged EXEC |
| `ping <IP>` | Tests IP connectivity | EXEC mode (`>` or `#`) |
| `do show ip interface brief` | Same as above, run from config mode | Any configuration mode |
| `do show ip route` | Same as above, run from config mode | Any configuration mode |
| `do ping <IP>` | Ping from config mode | Any configuration mode |

> 💡 `do` lets you run EXEC/privileged commands without leaving configuration mode.
> 

### 🧬 Conceptual Encapsulation Sequences (not CLI)

- **IPsec site-to-site:** Original IP packet → encrypt → IPsec encapsulation + new outer IP header
- **GRE over IPsec:** Original IP packet → GRE header + new IP header → encrypt → IPsec header + new outer IP header
- **MPLS:** L2 header → **MPLS label** → L3 IP header

---

# 📝 PART 4 — Lesson Quiz

### ❓ Question 1 — Leased-line bandwidth

**Which leased-line standard provides 1.544 Mbps?**

- A. E1
- B. T1
- C. T2
- D. E2

✅ **Answer: B — T1.** T1 = 1.544 Mbps. Know the T/E series, don't memorize every speed.

### ❓ Question 2 — MPLS router roles

**Which router type does NOT need to run MPLS?**

- A. PE
- B. P
- C. CE
- D. Both PE and P

✅ **Answer: C — CE.** PE and P routers run MPLS in the provider network.

### ❓ Question 3 — MPLS VPN type

**Which MPLS VPN type allows CE routers to form OSPF peerings directly with each other?**

- A. Layer 2 MPLS VPN
- B. Layer 3 MPLS VPN
- C. Layer 2.5 MPLS VPN
- D. Both Layer 2 and Layer 3 MPLS VPNs

✅ **Answer: A — Layer 2 MPLS VPN.** The provider acts like a big switch. In L3 MPLS VPN, CE peers with PE. "Layer 2.5" describes label placement, not a VPN type.

### ❓ Question 4 — Internet access technology

**Which Internet access technology uses already-installed telephone lines?**

- A. Cable Internet
- B. DSL
- C. Fiber Ethernet
- D. CATV

✅ **Answer: B — DSL.** Cable Internet uses CATV lines.

### ❓ Question 5 — Multicast over a VPN tunnel

**Which protocol can be combined with IPsec for flexibility in forwarding multicast traffic through a tunnel?**

- A. TLS
- B. HDLC
- C. GRE
- D. PPP

✅ **Answer: C — GRE.** GRE carries multicast/broadcast but doesn't encrypt; GRE over IPsec gives flexibility plus encryption.