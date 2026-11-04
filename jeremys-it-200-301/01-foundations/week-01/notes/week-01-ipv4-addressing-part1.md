# 🌐💻 IPv4 Addressing — Part 1

**📅 Date:** 10.24.2026

---

## 🎯 Big Picture: Layer 2 → Layer 3

| Layer | Focus | Addressing |
| --- | --- | --- |
| **Layer 2** | Switching | MAC Addresses |
| **Layer 3** | Routing | IP Addresses |

> **💡 Key Idea:** Layer 3 handles **logical addressing** and **path selection** to allow communication between **different networks**.
> 

---

## 🧱 IPv4 Basics

- **32-bit logical address**, split into 4 sections called **Octets** (8 bits each).
- Written in **Dotted Decimal Notation** for human readability (e.g., `192.168.1.1`).
- **Binary (Base 2):** Computers read IPs in 1s and 0s.
- **Decimal (Base 10):** Humans read IPs in numbers 0–255.

> **📌 Octet Value:** An 8-bit octet can hold a maximum value of **255** (all 1s: `11111111`).
> 

---

## 🔀 Network Portion vs. Host Portion

Every IP address is split into two parts:

1. **Network Portion** → Identifies the specific network. *(Like a street name)*
2. **Host Portion** → Identifies the specific device on that network. *(Like a house number)*

### How do we know the split?

| Method | Description | Example |
| --- | --- | --- |
| **Prefix Length** | Shorthand for number of network bits | `/24` = 24 bits for network |
| **Netmask (Subnet Mask)** | Dotted decimal representation of prefix | `/24` = `255.255.255.0` |

### 📝 Example: `192.168.1.10 /24`

- **Network portion:** `192.168.1`
- **Host portion:** `.10`

---

## 🏷️ IPv4 Address Classes (Classful)

| Class | First Octet Range | Default Prefix | Typical Use |
| --- | --- | --- | --- |
| **A** | 0 – 127 | `/8` | Massive networks |
| **B** | 128 – 191 | `/16` | Medium networks |
| **C** | 192 – 223 | `/24` | Small networks |

> **⚠️ Note:** Modern networks use **CIDR (Classless)**, but you **must memorize these defaults** for the CCNA.
> 

---

## 📍 Special Addresses

### 🏠 Network Address

- The **very first address** in a network (all host bits = `0`).
- **Cannot be assigned** to a device.
- **Example:** `192.168.1.0 /24`

### 📢 Broadcast Address

- The **very last address** in a network (all host bits = `1`).
- Sends data to **all devices** in the network.
- **Example:** `192.168.1.255 /24`

### 🔄 Loopback Address

- `127.x.x.x` (usually `127.0.0.1`).
- Used to test the **local device's TCP/IP stack** (localhost).

### 📡 Multicast Address (Class D)

- **Range:** `224.0.0.0` – `239.255.255.255`
- Used for **one-to-many** communication. A single packet is sent to a specific group of devices that have joined that group.
- **Example:** `224.0.0.5` (OSPF routers)

### 🧪 Experimental Address (Class E)

- **Range:** `240.0.0.0` – `255.255.255.255`
- Reserved by the IETF for **experimental or future use**.
- Non-routable and typically dropped by modern devices.
- **Example:** `240.0.0.1`

---

## 🧪 Practice Quiz: Binary & Decimal Conversions

### 🔢 Binary → Dotted Decimal

> **💡 Tip:** Remember the place values: **128 | 64 | 32 | 16 | 8 | 4 | 2 | 1**
> 

| Binary | Dotted Decimal |
| --- | --- |
| `00111111 00111000 11100111 00010011` | **63.56.231.19** |
| `11110011 01111111 01100010 00000001` | **243.127.98.1** |
| `01101111 00000110 01011001 11000111` | **111.6.89.199** |
| `11001111 11000110 00101111 01001100` | **207.198.47.76** |
| `01100100 11001001 00100001 11111101` | **100.201.33.253** |

### 🔢 Decimal → Binary

| Dotted Decimal | Binary |
| --- | --- |
| `88.46.90.91` | `01011000` | `00101110` | `01011010` | `01011011` |
| `221.234.246.163` | `11011101` | `11101010` | `11110110` | `10100011` *(Fixed last octet)* |
| `3.41.143.222` | `00000011` | `00101001` | `10001111` | `11011110` |
| `10.200.231.91` | `00001010` | `11001000` | `11100111` | `01011011` |
| `248.87.255.152` | `11111000` | `01010111` | `11111111` | `10011000` |

---

## 🧠 Quick Memory Hooks

### 🔍 First Octet Class Check

| Class | First Octet Range |
| --- | --- |
| **A** | 1 – 126 |
| **B** | 128 – 191 |
| **C** | 192 – 223 |

### 🧮 Host Math

> **Formula:** Total usable hosts in a subnet = **2^(host bits) − 2**
> 
> 
> *Subtract 2 for the Network and Broadcast addresses.*
> 

### 🪄 The "Magic Number" Technique

> **How to quickly find the network address:**
> 
> 
> Subtract the subnet mask octet from **256**.
> 
> **Example:** Mask `255.255.255.192` → `256 − 192 = 64`
> 
> Network addresses go in increments of **64**: `.0, .64, .128, .192`
> 

### 🔄 Prefix ↔ Netmask Quick Reference

| Prefix | Netmask |
| --- | --- |
| `/8` | `255.0.0.0` |
| `/16` | `255.255.0.0` |
| `/24` | `255.255.255.0` |
| `/25` | `255.255.255.128` |
| `/26` | `255.255.255.192` |
| `/27` | `255.255.255.224` |
| `/28` | `255.255.255.240` |
| `/29` | `255.255.255.248` |
| `/30` | `255.255.255.252` |

---

## 📋 Cheat Sheet Summary

| Concept | Key Fact |
| --- | --- |
| IPv4 Size | 32 bits = 4 octets × 8 bits |
| Max Octet Value | 255 (`11111111`) |
| Network Address | First address (host bits = 0) — **not assignable** |
| Broadcast Address | Last address (host bits = 1) — **sends to all** |
| Loopback | `127.0.0.1` — tests local TCP/IP stack |
| Multicast (Class D) | `224.0.0.0` – `239.255.255.255` — one-to-many |
| Experimental (Class E) | `240.0.0.0` – `255.255.255.255` — reserved |
| Usable Hosts | `2^(host bits) − 2` |
| Magic Number | `256 − mask octet` = network increment |