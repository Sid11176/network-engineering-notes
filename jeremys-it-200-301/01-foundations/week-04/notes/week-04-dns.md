# 🌐 DNS (Domain Name System) — CCNA Notes

---

## 🎯 Purpose of DNS

DNS **resolves human-readable names → IP addresses**. “Resolve” simply means “convert”.

| Without DNS | With DNS |
| --- | --- |
| You must remember `142.250.190.46` | You type `google.com` |
| Machines use IP addresses | Humans use names |
- Your device asks a **DNS server** for the IP address of the name you entered.
- The DNS server your device uses can be **manually configured** or **learned via DHCP**.

---

## 📜 DNS Record Types (CCNA-Level)

| Record | Maps | Example |
| --- | --- | --- |
| **A** | Name → **IPv4** address | `youtube.com → 172.217.25.110` |
| **AAAA** (“quadruple A”) | Name → **IPv6** address | `youtube.com → 2607:f8b0:...` |
| **CNAME** | Name → **another name** (canonical name) | May appear in DNS cache, pointing to the “real” A record |

> 🔑 **Memory hook:** **A** = **A**ddress (v4). **AAAA** = **A**ddress (v6, longer).
> 

---

## ⚙️ DNS Transport Layer — TCP vs. UDP

| Condition | Protocol | Port |
| --- | --- | --- |
| **Standard** queries/responses (≤ 512 bytes) | **UDP** | **53** |
| Messages **greater than 512 bytes** | **TCP** | **53** |

Standard DNS lookups almost always use **UDP**. TCP is reserved for larger messages (e.g., zone transfers).

---

## 🖥️ Windows Command Prompt Commands (Client Verification)

These commands are useful for CCNA because **“verify IP parameters for client OS”** is an exam topic.

| Command | What It Does | Why It Matters |
| --- | --- | --- |
| `ipconfig` | Shows basic IP info (IP, mask, gateway) | ❌ **Does NOT show the DNS server** |
| `ipconfig /all` | Full TCP/IP configuration | ✅ **Shows the configured DNS server** (plus MAC, DHCP info) |
| `ipconfig /displaydns` | Displays the **local DNS cache** | Shows recently resolved records (A, AAAA, CNAME) |
| `ipconfig /flushdns` | **Clears** the DNS resolver cache | Forces fresh queries on next access |
| `nslookup <name>` | Queries the DNS server for a record | ✅ Also reveals which DNS server the PC is using |

> 🔁 The **DNS cache** prevents repeated queries for the same name, saving network traffic.
> 

---

## 📄 The `hosts` File (Pre-DNS Legacy)

A simple local text file mapping hostnames → IP addresses.

- **Windows path:** `C:\Windows\System32\drivers\etc\hosts`
- Format: `IP_address hostname` (space-separated)
- Used **before DNS existed**; still present today for small/local overrides.
- **Not DNS** — it’s a manual alternative.

---

# 🛠️ Cisco IOS DNS Configuration

> ⚠️ **Critical exam point:** For hosts to use DNS, **no DNS configuration is required on the router**. The router simply forwards DNS packets like any other traffic.
> 

A Cisco router *can* act as a DNS **server** and/or **client**, but this is rarely done in production (internal DNS is usually Windows/Linux).

---

## 🖥️ Router as a DNS Server

### Syntax (Global Config Mode)

```
ip dns server
ip host <hostname> <ip-address>
ip name-server <dns-server-ip>
ip domain lookup        ! enabled by default
```

| Command | Purpose |
| --- | --- |
| `ip dns server` | Enables the router to **act as a DNS server** |
| `ip host <name> <ip>` | Builds a **static host table** entry (permanent) |
| `ip name-server <ip>` | Configures an **external DNS server** for the router to query when it lacks a record |
| `ip domain lookup` | Enables DNS queries on the router; **on by default** |

> 🧠 Older IOS syntax: `ip domain-lookup` (with hyphen). **Both versions are supported** — know both for the exam.
> 

### 🔄 Server + Client Behavior (Forwarding)

1. PC queries **R1** for a name.
2. If R1 **has the record** (via `ip host`), it replies directly.
3. If R1 **does NOT have the record**, it acts as a **client**, queries the external `ip name-server`, then relays the answer back to the PC.

---

## 📥 Router as a DNS Client

Only **two commands** are needed:

```
ip name-server <dns-server-ip>
ip domain lookup          ! enabled by default
```

- After this, the router can `ping` names (e.g., `ping youtube.com`).
- The router does **not** answer DNS queries from other devices unless `ip dns server` is also configured.

---

## 🏷️ Optional: Default Domain Name

```
ip domain name <domain-name>
! older syntax: ip domain-name <domain-name>
```

- Defines a **default domain** appended to unqualified hostnames.
- Example: `ping PC1` becomes `ping PC1.jeremysitlab.com`.
- **Required for SSH** (covered in a later lesson).

---

## 📊 Viewing DNS Information on Cisco IOS

```
show hosts
```

| Entry Type | Flag | Nature |
| --- | --- | --- |
| Manually configured (`ip host`) | `perm` | **Permanent** |
| Learned via DNS | `temp` | **Temporary** (will expire and be re-learned) |

---

# 🧠 Key Takeaways & Exam Traps

| ✅ Do Remember | ❌ Common Mistake |
| --- | --- |
| DNS resolves **names → IPs** (A for v4, AAAA for v6). | Thinking **AAA** maps to IPv6 — it’s **AAAA**. |
| Standard DNS uses **UDP/53**; TCP for **>512 bytes**. | Assuming DNS always uses UDP — large messages use TCP. |
| Routers **forward** DNS without any config. | Believing `ip dns server` is required for clients to use an external DNS. |
| `ipconfig /all` and `nslookup` both reveal the **DNS server address**. | Thinking `ipconfig` (alone) shows the DNS server — it does **not**. |
| A router can be **server AND client simultaneously**. | Assuming these roles are mutually exclusive. |

---

## 📝 Quick Quiz (Self-Check)

1. **Which Windows commands reveal the configured DNS server?** *(Select two)*
→ `ipconfig /all` and `nslookup`.
2. **Which record type maps a name to IPv6?**
→ **AAAA** (quadruple A).
3. **Does a router need DNS config to forward DNS traffic between a PC and 8.8.8.8?**
→ **No.** It simply routes the packets.
4. **Which Cisco IOS command shows DNS-cached and manually configured host entries?**
→ `show hosts`.
5. **Which protocol lets hosts automatically learn their DNS server address?**
→ **DHCP**.

---

## 🖥️ Windows Command Prompt (Client Verification)

| Syntax | Used For |
| --- | --- |
| `ipconfig` | Displays basic IP parameters (IP address, subnet mask, default gateway) — **not** the DNS server |
| `ipconfig /all` | Displays the **full** TCP/IP configuration, including the configured **DNS server** address |
| `ipconfig /displaydns` | Displays the local **DNS cache** (recently resolved A, AAAA, CNAME records) |
| `ipconfig /flushdns` | **Clears** the DNS resolver cache, forcing fresh queries on the next access |
| `nslookup <hostname>` | Queries the DNS server for a name’s IP address; also reveals **which DNS server** the PC is using |
| `ping <hostname>` | Tests connectivity using a name; the name is resolved via DNS/hosts file first |

---

## 🛠️ Cisco IOS — DNS Server Configuration (Global Config Mode)

| Syntax | Used For |
| --- | --- |
| `ip dns server` | Enables the router to **act as a DNS server** (responds to client queries if it has the record) |
| `ip host <hostname> <ip-address>` | Creates a **static/permanent** host table entry mapping a name to an IP |
| `ip name-server <dns-server-ip>` | Configures an **external DNS server** for the router to query when it lacks a record |
| `ip domain lookup` | Enables the router to **perform DNS queries** (enabled by default); older form: `ip domain-lookup` |
| `ip domain name <domain-name>` | Sets a **default domain** appended to unqualified hostnames (required for SSH); older form: `ip domain-name` |

---

## 🛠️ Cisco IOS — DNS Client Configuration

| Syntax | Used For |
| --- | --- |
| `ip name-server <dns-server-ip>` | Points the router to the DNS server it should query as a **client** |
| `ip domain lookup` | Allows the router to send DNS queries (on by default) — e.g., `ping youtube.com` |

---

## 📊 Cisco IOS — Verification

| Syntax | Used For |
| --- | --- |
| `show hosts` | Displays **both** manually configured (`perm`) and DNS-learned/cached (`temp`) host entries |

---

## 🔑 Protocol & Port Reference

| Item | Value |
| --- | --- |
| DNS port | **53** (both UDP and TCP) |
| Standard DNS queries/responses | **UDP**, ≤ **512 bytes** |
| Large DNS messages | **TCP**, **> 512 bytes** |
| **A** record | Name → **IPv4** address |
| **AAAA** record | Name → **IPv6** address |
| **CNAME** record | Name → **another name** |