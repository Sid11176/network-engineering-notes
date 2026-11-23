# 🔐 CCNA Day 57 — Wireless Security

> 🎥 Jeremy's IT Lab · CCNA 200-301 · Exam topics: **1.11.d** Encryption · **5.9** Describe wireless security protocols (WPA, WPA2, WPA3)
> 
> 
> 📌 Content is *wide but shallow*: know the **differences** between methods.
> 

---

## 🧠 1. Why Wireless Security Matters

- 📡 Wireless signals aren't contained in a wire → **any device in range can receive traffic**.
- Wired LANs usually encrypt only over untrusted networks (Internet); wireless **must encrypt client ↔ AP traffic**.
- **3 core concepts:** 🪪 Authentication · 🔒 Encryption · ✅ Integrity

### 🪪 Authentication

- Verifies identity of a user/device. **Clients must authenticate before associating** with an AP.
- Methods: password, username + password, or **digital certificates**.
- 👥 Corporate SSID = trusted users only. **Guest SSID** = looser authentication, Internet-only access.
- Ideally the **client also authenticates the AP** → avoids **rogue/malicious AP** and **man-in-the-middle** attacks.

### 🔒 Encryption

- Scrambles data so only sender and intended recipient can read it. Both sides must use the **same protocol**.
- All devices on the WLAN use the same protocol, but **each client has a unique key** (only the AP can decrypt it).
- 👥 **Group key**: used by the AP to encrypt traffic to **all** clients (all clients hold a copy).

### ✅ Integrity (MIC — Message Integrity Check)

1. Sender calculates a **MIC** and attaches it to the message
2. Sender **encrypts** message + MIC and sends
3. Receiver **decrypts**, independently calculates its own MIC (same protocol)
4. ✔️ Match → accepted · ❌ Different → **message discarded**
- Note: a MIC *detects* tampering; it doesn't prevent it.

---

## 🔑 2. Authentication Methods (7 total)

### 🟥 Legacy (insecure)

| Method | Key Points |
| --- | --- |
| **Open Authentication** | Client requests, AP accepts. **No credentials.** Not secure alone, but still used with other methods (e.g., public Wi-Fi + web login portal) |
| **WEP** (Wired Equivalent Privacy) | Provides **encryption + authentication**. Uses **RC4**. **Shared-key**. Keys: **40 or 104 bits** + **24-bit IV** → **64 / 128 bits** total. ❌ **Easily cracked regardless of key length, never use** |

**🧩 WEP authentication flow (challenge-response):**

1. AP sends a **challenge phrase**
2. Client encrypts it with its WEP key and sends it back
3. AP compares with its own encrypted result → match = authenticated (client knows the shared key)

> If WEP authentication isn't used, **Open auth** is used and WEP only encrypts.
> 

### 🟩 EAP (Extensible Authentication Protocol)

- **EAP = a framework**, not a single protocol. Specific protocols are **EAP methods** (LEAP, EAP-FAST, PEAP, EAP-TLS).
- Integrated with **802.1X** (port-based network access control: limits access until the client authenticates). Used in **both wired and wireless** networks.

### 👥 802.1X Entities ⭐

| Entity | Role | In a WLAN |
| --- | --- | --- |
| **Supplicant** | Device that wants to connect | Client (laptop) |
| **Authenticator** | Provides access to the network | **AP** (or the **WLC** in split-MAC architecture) |
| **Authentication Server (AS)** | Receives credentials, permits/denies | Usually a **RADIUS server** |

**Flow:** Client does **Open auth** to associate → but only **EAP traffic** is allowed → AS checks credentials → permit or deny. The WLC acts as the middleman.

### 📋 EAP Methods Compared

| Method | Developed by | How it works | Security |
| --- | --- | --- | --- |
| **LEAP** (Lightweight EAP) | Cisco | Username/password + **mutual** challenge phrases (both client and server) + **dynamic WEP keys** | ❌ Vulnerable, don't use |
| **EAP-FAST** (Flexible Auth via Secure Tunneling) | Cisco | **3 phases:** ① server generates **PAC** (protected access credential) and passes it to the client → ② PAC establishes a **TLS tunnel** → ③ client authenticated inside the tunnel | ✅ Secure |
| **PEAP** (Protected EAP) | — | Server shows a **digital certificate** → authenticates the server and builds the **TLS tunnel** → client authenticated inside tunnel (e.g., **MS-CHAP**) | ✅ Secure, easier to deploy |
| **EAP-TLS** (Transport Layer Security) | — | **Certificate on the AS AND every client**. Mutual auth with certs; no need to authenticate the client inside the tunnel (tunnel is still used to exchange encryption key info) | 🏆 **Most secure**, but complex to implement |

> 🔑 **EAP-FAST = PAC · PEAP = server certificate only · EAP-TLS = certificates on both sides**
> 
> 
> Many enterprises choose **PEAP** over EAP-TLS because of the effort of deploying client certificates.
> 

---

## 🔒 3. Encryption & Integrity Methods

> Order of development: **WEP → TKIP → CCMP → GCMP** (each newer = more secure)
> 

| Protocol | Used in | Encryption | Integrity (MIC) | Notes |
| --- | --- | --- | --- | --- |
| **WEP** | Original 802.11 | RC4 | — | ❌ Insecure |
| **TKIP** (Temporal Key Integrity Protocol) | **WPA** | WEP-based | Yes (MIC) | Temporary fix on existing WEP hardware |
| **CCMP** (Counter/CBC-MAC Protocol) | **WPA2** | **AES counter mode** | **CBC-MAC** | Requires hardware support |
| **GCMP** (Galois/Counter Mode Protocol) | **WPA3** | **AES counter mode** | **GMAC** | 🏆 Most secure & efficient, higher throughput |

### 🛠️ TKIP Features (just know it's "a more secure WEP")

- **MIC** for integrity
- **Key mixing** → unique WEP key per frame
- **IV doubled** 24 → 48 bits (harder brute force)
- MIC includes **sender MAC** (proves sender identity)
- **Timestamp** and **sequence number** → protect against **replay attacks** (re-sending previously sent frames)

### 🧮 CCMP & GCMP

- **AES** = the most secure encryption available, used worldwide. Counter mode chosen for **speed**.
- **CBC-MAC** (CCMP) and **GMAC** (GCMP) are both *kinds of MIC*, no need to know how they work internally.

---

## 🛡️ 4. WPA Certifications (Wi-Fi Alliance)

- Packages protocols into **standard sets**; devices tested in authorized labs and certified.
- The first is just **"WPA"**, not "WPA1".

### 🔀 Two Authentication Modes (all 3 WPAs)

| Mode | Method | Typical use |
| --- | --- | --- |
| 🏠 **Personal** | **PSK** (Pre-Shared Key), the Wi-Fi password | SOHO / home networks |
| 🏢 **Enterprise** | **802.1X + AS** (any EAP method: PEAP, EAP-TLS, etc.) | Corporate networks |
- 🔐 The PSK **isn't sent over the air**: a **4-way handshake** is used, and the PSK generates the encryption keys.

### 📊 WPA vs WPA2 vs WPA3

|  | **WPA** | **WPA2** | **WPA3** (2018) |
| --- | --- | --- | --- |
| Encryption + MIC | **TKIP** | **CCMP** | **GCMP** |
| Authentication | 802.1X/EAP *or* PSK | 802.1X/EAP *or* PSK | 802.1X/EAP *or* PSK |
| Extras | — | PMF optional | **PMF mandatory**, **SAE**, **Forward secrecy** |

### 🆕 WPA3 Extras

- **PMF (Protected Management Frames):** protects 802.11 management frames from **eavesdropping and forging** (optional in WPA2, **mandatory** in WPA3)
- **SAE (Simultaneous Authentication of Equals):** protects the **4-way handshake** in personal mode
- **Forward secrecy:** captured frames can't be decrypted later

---

## ⌨️ 5. Syntax / Commands Used

> ℹ️ **No CLI commands were used in this lesson.** Day 57 is purely conceptual; wireless security configuration (WLC GUI/CLI) comes in later videos.
> 

| 🔑 Syntax | 📝 Description | 🖥️ Command mode |
| --- | --- | --- |
| — | No CLI syntax introduced | N/A (theory only) |

---

## ✅ 6. Quick Memory Cheat Sheet

- 🔢 Evolution: **WEP → TKIP → CCMP → GCMP**
- 🛡️ **WPA = TKIP · WPA2 = CCMP · WPA3 = GCMP**
- 🧬 CCMP = **AES-CTR + CBC-MAC** · GCMP = **AES-CTR + GMAC**
- 🏆 Most secure auth: **EAP-TLS** · Easier alternative: **PEAP**
- 🎫 **EAP-FAST = PAC** · **PEAP = server cert** · **EAP-TLS = both certs**
- 👥 802.1X: **Supplicant** (client) · **Authenticator** (AP/WLC) · **AS** (RADIUS)
- 🏠 Personal = PSK · 🏢 Enterprise = 802.1X
- 🔁 TKIP counters replay attacks with timestamp + sequence number
- 🆕 WPA3: **PMF** (mandatory), **SAE** (4-way handshake), **forward secrecy**

---

## ❓ 7. Quiz

**Q1.** What does **GMAC** provide to a secure wireless connection?

> ✅ **MIC (Message Integrity Check)**: used in GCMP, part of WPA3.
> 

**Q2.** Which are part of the **802.1X** authentication architecture? *(select three)*

> ✅ **Supplicant**
> 
> 
> ✅ **Authenticator**
> 
> ✅ **Authentication Server**
> 
> (Client = supplicant, AP/WLC = authenticator, RADIUS = AS)
> 

**Q3.** Which encryption and integrity method is considered **most secure**?

> ✅ **GCMP** (order of development: WEP → TKIP → CCMP → GCMP). Use it whenever hardware supports it.
> 

**Q4.** Which method requires a **certificate on both the supplicant and the AS**?

> ✅ **EAP-TLS** (PEAP only requires a certificate on the AS)
> 

**Q5.** Which **WPA3** feature protects the **4-way handshake** in personal mode?

> ✅ **SAE** (Simultaneous Authentication of Equals)
> 

**Q6 (Boson ExSim).** Which of the following is used by **WEP** to provide encryption? *(Select the best answer.)*

> ✅ **RC4**
> 
> 
> ❌ GCMP is used in WPA3, CCMP is used in WPA2, and AES is the algorithm inside CCMP and GCMP, not WEP.
>