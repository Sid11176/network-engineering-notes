# 📘 DHCP (Dynamic Host Configuration Protocol) — CCNA Notes

---

## 🎯 Purpose of DHCP

- Allows hosts to **automatically & dynamically** learn network configuration:
    - IP address, subnet mask, default gateway, DNS server, etc.
- Eliminates manual/static configuration
- **Used for client devices** (PCs, phones, workstations)
- **Servers/routers** usually manually configured (need fixed IPs)
- Small networks: router acts as DHCP server
- Large networks: centralized Windows/Linux DHCP server

---

## 🔄 DORA — The 4-Message DHCP Process

| Step | Message | Direction | Type |
| --- | --- | --- | --- |
| 1️⃣ | **Discover** | Client → Server | **Broadcast** |
| 2️⃣ | **Offer** | Server → Client | Broadcast or **Unicast** |
| 3️⃣ | **Request** | Client → Server | **Broadcast** |
| 4️⃣ | **Ack** | Server → Client | Broadcast or **Unicast** |

> 💡 **DORA** = **D**iscover, **O**ffer, **R**equest, **A**ck
> 

**Extra message:**

- **Release** — Client → Server (Unicast) — gives up IP early

### 🔍 Message Details

- **Discover** 🗣️: Broadcast MAC (all Fs), Source IP `0.0.0.0`, Dest IP `255.255.255.255`. Client may request a previously-used IP (Option 50).
- **Offer** 📬: Server offers IP + gateway + DNS + lease time. Sent unicast if client requests (BootP flags = 0000).
- **Request** 🙋: Broadcast so ALL DHCP servers see which offer was accepted (Option 54 = selected server IP).
- **Ack** ✅: Confirms client may use the IP. Client finally configures its interface.

### 🔌 Ports

- **DHCP Server**: UDP **67**
- **DHCP Client**: UDP **68**
- Source 68 → Dest 67 (client→server)
- Source 67 → Dest 68 (server→client)

> ⚠️ **BootP flags** field: `0000` = unicast, `broadcast` flag set = broadcast
> 

---

## 🔁 DHCP Relay Agent

**Problem:** Centralized DHCP server won't receive broadcast Discover messages (routers don't forward broadcasts).

**Solution:** Configure router as **DHCP Relay Agent** — forwards client broadcasts to remote DHCP server as **unicast**.

### Process:

1. PC1 broadcasts **Discover** → R1 receives
2. R1 relays to SRV1 (source = R1's G0/1 IP, dest = SRV1 IP) — **unicast**
3. SRV1 sends **Offer** → R1 → PC1
4. PC1 broadcasts **Request** → R1 relays → SRV1
5. SRV1 sends **Ack** → R1 → PC1 → PC1 configures IP

> ⚠️ Ensure R1 has a route to the DHCP server (static route or OSPF)
> 

---

## ⚙️ Cisco IOS Configuration

### 1️⃣ Router as DHCP Server

```
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp pool POOL_NAME
 network 192.168.1.0 /24
 dns-server 8.8.8.8
 domain-name jeremysitlab.com
 default-router 192.168.1.1
 lease 0 5 30
```

### 2️⃣ Router as DHCP Relay Agent

```
interface g0/1
 ip helper-address 192.168.10.10
```

**Verification:**

```
show ip interface g0/1
```

### 3️⃣ Router as DHCP Client

```
interface g0/1
 ip address dhcp
```

**Verification:**

```
show ip interface g0/1
```

> Should show: *"address determined by DHCP"*
> 

---

## 🖥️ Windows Commands

| Command | Purpose |
| --- | --- |
| `ipconfig /all` | Show full IP config (DHCP enabled, lease info, etc.) |
| `ipconfig /release` | Release DHCP-learned IP (sends **Release**) |
| `ipconfig /renew` | Request new IP (sends **Discover**) |

---

## ❓ Quiz Questions

### Q1: What is the correct order of messages when a DHCP client gets an IP?

**Answer: B — Discover, Offer, Request, Ack** (DORA 🐕)

### Q2: Which Windows command causes a PC to broadcast a DHCP Discover?

**Answer: D — `ipconfig /renew`**

- `ipconfig /release` → sends Release (not Discover)
- A & B are not real commands

### Q3: Examine DHCP Offer message — what destination IP did SRV1 use?

**Answer: D — `255.255.255.255`**

- Because **BootP broadcast flag** is set (not unicast)
- Server will broadcast to client

### Q4: Which DHCP messages can be sent using unicast? (select all)

**Answer: A, C, E**

- **A — DHCP Ack** ✅ (server→client, can be unicast)
- **C — DHCP Release** ✅ (always unicast)
- **E — DHCP Offer** ✅ (server→client, can be unicast)
- B (Discover) & D (Request) → **always broadcast**

### Q5: When would you configure a router as a DHCP relay agent?

**Answer: A** — When router is **not** a DHCP server, clients exist in its LAN, and **no other DHCP server** is in that LAN.

### 🎁 Bonus (Boson ExSim)

**Scenario:** DHCP server on NetworkB, clients on NetworkA, routers don't run DHCP.
**Answer: F — `ip helper-address 10.10.3.5` on RouterA**

- RouterA is relay agent (closest to clients)
- IP = DHCP server's address

---

## 📋 Command Reference Cheat Sheet (with explanations)

### 🖥️ Windows Commands

| Command | What It Does |
| --- | --- |
| `ipconfig /all` | Displays full IP configuration for all adapters — shows whether DHCP is enabled, assigned IP, subnet mask, default gateway, DNS server, DHCP server address, and **lease obtained/expires** times |
| `ipconfig /release` | Sends a **DHCP Release** message to the server, giving up the current DHCP-learned IP address so it can be reassigned to another device |
| `ipconfig /renew` | Sends a **DHCP Discover** message to obtain a new (or the same preferred) IP address from any reachable DHCP server |

### ⚙️ Cisco IOS — DHCP Server Configuration

| Command | What It Does |
| --- | --- |
| `ip dhcp excluded-address <low> <high>` | Reserves a range of addresses that the DHCP server will **never assign** to clients — used for servers, routers, printers, etc. that need static IPs. (First IP = low end, second IP = high end of range) |
| `ip dhcp pool <name>` | Creates a **DHCP pool** (a subnet of assignable addresses + config info). You need one pool per network the router serves. Enters DHCP config mode |
| `network <net> <mask|prefix>` | Defines the **range of addresses** the pool can assign to clients. Accepts either dotted mask (`255.255.255.0`) or prefix (`/24`) |
| `dns-server <ip>` | Tells DHCP clients which **DNS server** to use for name resolution |
| `domain-name <name>` | Tells DHCP clients which **domain** they belong to (appended to hostnames for DNS queries) |
| `default-router <ip>` | Tells DHCP clients which **default gateway** to use (usually the router's own LAN interface IP) |
| `lease <days> <hours> <minutes>` | Sets how long clients may keep the assigned IP before it expires (e.g., `lease 0 5 30` = 5h 30m). Use `lease infinite` for permanent — not recommended |
| `show ip dhcp binding` | Lists all DHCP clients currently holding a lease — shows IP, MAC address, lease expiration, and binding type (manual vs. automatic) |

### ⚙️ Cisco IOS — DHCP Relay Agent Configuration

| Command | What It Does |
| --- | --- |
| `interface <int>` | Enters interface config mode for the interface **facing the DHCP clients** (the LAN side) |
| `ip helper-address <dhcp-server-ip>` | Configures the router to **forward DHCP broadcasts** received on this interface to the specified DHCP server as **unicast** — this is the DHCP relay agent function |
| `show ip interface <int>` | Verifies config — the **"helper address"** field should show the DHCP server's IP |

### ⚙️ Cisco IOS — DHCP Client Configuration

| Command | What It Does |
| --- | --- |
| `interface <int>` | Enters interface config mode for the interface that should obtain an IP via DHCP |
| `ip address dhcp` | Makes the interface a **DHCP client** — it will broadcast a Discover message and configure its IP from a DHCP server's response |
| `show ip interface <int>` | Verifies config — should display **"address determined by DHCP"** |

---

## 🔑 Key Takeaways

- ✅ DHCP = automatic IP configuration for clients
- ✅ **DORA** = Discover → Offer → Request → Ack
- ✅ Server port **67**, Client port **68**
- ✅ Discover & Request = **always broadcast**
- ✅ Offer & Ack = **broadcast or unicast** (client decides via BootP flags)
- ✅ Release = **always unicast**
- ✅ Relay agent fixes broadcast-not-crossing-routers problem
- ✅ `ip helper-address` = one command to configure relay
- ✅ `ip address dhcp` = make router a DHCP client