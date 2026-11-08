# 📘 CCNA NAT Notes — Static NAT, Dynamic NAT, PAT

## 🧭 Core Concepts

- **Inside local**: private/source IP inside the network.
- **Inside global**: public/translated IP used outside.
- **Static NAT**: manual one-to-one mapping.
- **Dynamic NAT**: router creates one-to-one mappings automatically, temporarily.
- **PAT / NAT Overload**: many-to-one; translates IP **and port if needed** so many hosts share one public IP. 🌍

---

## 🔒 Static NAT Review

- Manually maps one private IP to one public IP.
- Outbound: router translates **source** IP.
- Reply: router translates **destination** IP back.
- ✅ **Bidirectional**: external hosts can initiate to the inside global IP, and the router translates it to the inside local IP.
- Static mappings are **permanent**.

---

## 🔄 Dynamic NAT

- Router dynamically maps inside local → inside global **as needed**.
- No manual “this IP to that IP” mapping.
- **ACL identifies traffic to translate**:
    - Permit = translate source IP.
    - Deny = **do not translate** — but does **not** mean drop, unless the ACL is applied elsewhere with `ip access-group`.
- **NAT pool** defines available inside global addresses.
- Mappings are still **one-to-one**, but **temporary**.
- If no pool address is available → **NAT pool exhaustion** → packet is **dropped** until an address frees up.
- Dynamic entries time out if unused; can also clear manually.
- Default dynamic mapping timeout: **24 hours**; UDP/ICMP translation entries ~1 minute; timer resets when used. Extended-entry details are not CCNA-critical.
- Verify with:
    - `show ip nat translations`
    - `show ip nat statistics` — confirms ACL-to-pool dynamic mapping.

---

## 🚪 PAT / NAT Overload

- **Port Address Translation = NAT Overload**.
- Translates **IP address + port number if necessary**.
- Many inside hosts share **one public IP** at the same time.
- Router tracks flows using unique **Layer 4 port numbers**.
- If source ports are already unique, router may not need to translate the port.
- Most widely used NAT type; best for preserving public IPv4 addresses.
- Two common config styles:
    1. Pool + `overload`
    2. Outside interface IP + `overload`
- With PAT, `show ip nat translations` does **not** show one-to-one dynamic mappings; it shows **many-to-one** translations.

---

## 🔧 Syntax Summary

| Syntax / Command | Where configured/executed | What it does |
| --- | --- | --- |
| `interface g0/1` → `ip nat inside` | Interface config on NAT router | Marks interface as NAT **inside** |
| `interface g0/0` → `ip nat outside` | Interface config on NAT router | Marks interface as NAT **outside** |
| `access-list 1 permit 192.168.0.0 0.0.0.255` | Global config on NAT router | ACL identifies traffic to translate; uses **wildcard mask**; `/24` = `0.0.0.255` |
| `ip nat pool POOL1 100.0.0.0 100.0.0.255 prefix-length 24` | Global config on NAT router | Defines inside global address pool; first/last must be in same subnet |
| `ip nat pool POOL1 100.0.0.0 100.0.0.255 netmask 255.255.255.0` | Global config on NAT router | Same as above using netmask option |
| `ip nat inside source static 192.168.0.167 100.0.0.1` | Global config on NAT router | Creates permanent one-to-one static NAT mapping |
| `ip nat inside source list 1 pool POOL1` | Global config on NAT router | Configures **dynamic NAT**: ACL 1 traffic uses POOL1 |
| `ip nat inside source list 1 pool POOL1 overload` | Global config on NAT router | Configures **PAT using a pool** |
| `ip nat inside source list 1 interface g0/0 overload` | Global config on NAT router | Configures **PAT using outside interface IP** |
| `show ip nat translations` | Privileged EXEC / EXEC | Displays active NAT translation entries |
| `show ip nat statistics` | Privileged EXEC / EXEC | Displays NAT stats, active translations, and dynamic ACL/pool mapping |
| `clear ip nat translation` | Privileged EXEC | Manually clears **dynamic** NAT translations; static remain |

📌 ACL note: In NAT, the ACL is only **selecting traffic**; it is not applied to an interface, so denied traffic is not translated but is not necessarily dropped.

---

# 🧪 Quiz

1. **Which NAT type best fulfills the goal of preserving public IPv4 addresses?**
    
    ✅ **D — NAT Overload / PAT.** Many hosts share one public IP via Layer 4 port tracking.
    
2. **Which dynamic NAT config translates `172.16.1.0/24` to addresses from `203.0.113.0/25`?**
    
    ✅ **B.**
    
    - A is wrong: pool range includes `203.0.113.255`, outside `/25`.
    - C is wrong: ACL uses subnet mask instead of **wildcard mask**.
3. **Dynamic NAT pool has 10 inside global addresses, all in use; another inside host sends a packet. What does R1 do?**
    
    ✅ **B — Discards the packet.** No available pool address = drop until one frees up.
    
4. **Which dynamic NAT config translates `10.0.1.0/27` to the router’s `G0/1` interface IP?**
    
    ✅ **A.** Only option with correct `/27` wildcard, correct `ip nat` command, and correct inside/outside interface assignment.
    
5. **ACL used for dynamic NAT/PAT denies `192.168.1.0/24`. What happens to those hosts’ packets?**
    
    ✅ **C — They will not be translated by R1.** The NAT ACL denies translation, but does not automatically drop the packets.
    
6. **⭐ Boson ExSim bonus:** HostA initiates HTTP to HostB through RouterA. Which `show ip nat translations` line is expected?
    
    ✅ **C.**
    
    - Inside local = HostA: `10.1.7.7`
    - Inside global = RouterA external/public IP: `203.0.113.62`
    - Outside local/global = HostB: `192.0.2.28`
    - A/D wrong inside local (`10.1.7.1`); B wrong inside global (`10.1.7.1`, RouterA internal).