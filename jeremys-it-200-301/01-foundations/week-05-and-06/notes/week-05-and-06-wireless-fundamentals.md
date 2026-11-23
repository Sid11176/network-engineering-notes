# 📡 CCNA Day 55 — Wireless Fundamentals

> 🎥 Jeremy's IT Lab · CCNA 200-301 · Exam topics: **1.1.d** Access Points · **1.11.a** Non-overlapping Wi-Fi channels · **1.11.b** SSID · **1.11.c** RF
> 

---

## 🧠 1. Wireless vs. Wired LANs

| Issue | Wired (switch) | Wireless |
| --- | --- | --- |
| Who receives frames? | Only intended recipient | **All devices in range** (like a hub) |
| Duplex | Full-duplex | **Half-duplex** |
| Collisions | Shouldn't occur | Possible → solved with **CSMA/CA** |
| Privacy | Encryption usually only over WAN/Internet | ⚠️ **Must encrypt even inside the LAN** |
| Regulation | None on cable | 🌍 Regulated by country/intl. bodies |
| Coverage | Cable length, EMI | Range + signal-quality factors |

### 🚦 CSMA/CA (Collision **Avoidance**) — wireless

1. Build frame
2. Listen: is channel free?
3. ❌ Busy → wait random time → listen again
4. ✅ Free → transmit
- 📌 **CSMA/CD** = wired, *detects & recovers* from collisions. **CSMA/CA** = wireless, *avoids* collisions before they happen.
- Optional RTS/CTS (Request/Clear To Send) exists, but **not needed for CCNA**.

### 🌫️ Factors Affecting Signal

| Factor | Meaning | Example |
| --- | --- | --- |
| **Absorption** | Material converts signal to heat → weaker | Wall |
| **Reflection** | Signal bounces off | Metal, elevators |
| **Refraction** | Wave bends in a medium of different speed | Glass, water |
| **Diffraction** | Wave bends around obstacle → blind spots | Object blocking PC |
| **Scattering** | Signal scatters in all directions | Dust, smog, uneven surfaces |
| **Interference** | Other devices on same channel | Neighbor's Wi-Fi |

---

## 📻 2. Radio Frequency (RF) Basics

- AC applied to an antenna → electromagnetic waves radiate out.
- **Amplitude** = max strength of the wave.
- **Frequency** = cycles per second (**Hz** → kHz → MHz → GHz → THz).
- **Period** = time of one cycle (4 Hz → 0.25 s).
- RF range: ~**30 Hz – 300 GHz** (visible light ≈ 400–790 THz).
- 📝 Wi-Fi uses parts of the **UHF** and **SHF** ranges.

### 📶 Wi-Fi Bands

| Band | Range | Pros | Cons |
| --- | --- | --- | --- |
| **2.4 GHz** | 2.4–2.4835 GHz | Longer reach, better wall penetration | ⚠️ More crowded → more interference |
| **5 GHz** | 5.150–5.825 GHz (4 sub-bands) | Less interference, **non-overlapping channels** | Shorter range |
| **6 GHz** | Added by **Wi-Fi 6 (802.11ax)** | — | Maybe on exam, just know it exists |

> 💡 Only memorize the two main bands (**2.4 & 5 GHz**), not exact ranges.
> 

### 🔢 Channels

- Each band is split into **channels**; **channel bonding** combines channels (not needed for CCNA).
- 2.4 GHz channels are **22 MHz wide and overlap** (e.g., Ch 1 = 2401–2423 MHz overlaps Ch 2–5).
- ⭐ **Use channels 1, 6, 11** in 2.4 GHz (non-overlapping) when multiple APs are adjacent. *(Outside North America other combos are possible, but remember 1-6-11 for CCNA.)*
- Single AP → any channel works.
- 5 GHz → channels don't overlap → easier.
- 🐝 **Honeycomb pattern**: AP coverage areas overlap, channels don't → full coverage, no co-channel interference.

---

## 📜 3. Wi-Fi Standards (IEEE 802.11)

- Wired = **IEEE 802.3** · Wireless = **IEEE 802.11** (first released 1997)
- **Wi-Fi** = trademark of the **Wi-Fi Alliance** (tests/certifies compliance & interoperability; not the IEEE). Technically 802.11 is the correct term, but both are used.
- Naming: **802.11n = Wi-Fi 4**, **802.11ac = Wi-Fi 5**, **802.11ax = Wi-Fi 6** (2019). ❌ No official Wi-Fi 1, 2, 3.
- 🎯 Jeremy recommends **memorizing each standard's name, frequency, and max theoretical data rate** (theoretical; real speeds are lower). Use the course flashcards.
- Devices may support one, some, or all standards → check before buying.
- Full Wi-fi and IEEE 802.11 generations please refer to [https://en.wikipedia.org/wiki/IEEE_802.11#Generations](https://en.wikipedia.org/wiki/IEEE_802.11#Generations)

---

## 🏷️ 4. Service Sets

> All devices in a service set share the same **SSID**.
> 

### 🔤 SSID vs BSSID

| Term | Meaning |
| --- | --- |
| **SSID** | Human-readable network name. Doesn't have to be unique (but best practice is unique) |
| **BSSID** | **MAC address of the AP's radio** → unique per AP/BSS |

### 3 Types of Service Sets

| Type | Description |
| --- | --- |
| **Independent (IBSS)** | Devices connect **directly**, no AP. "Ad hoc" (e.g., AirDrop). Not scalable |
| **Infrastructure** (BSS, ESS) | Clients connect via AP |
| **Mesh (MBSS)** | APs bridge traffic over wireless backhaul |

### 🏢 Infrastructure Service Sets

- **BSS (Basic Service Set)**: One AP + its clients. Clients **associate** with the AP and are called **clients / stations**. Clients **never talk directly**; traffic always goes **through the AP**, even if in range.
- **BSA (Basic Service Area)**: the **physical area** of AP coverage. 👉 **BSS = group of devices; BSA = physical area.**
- **ESS (Extended Service Set)**: multiple BSSs connected by a wired network (switch) to cover larger areas.
    - ✅ **Same SSID** across BSSs
    - ✅ **Unique BSSID** per BSS
    - ✅ **Different (non-overlapping) channels** (e.g., Ch 1 and Ch 6)
    - 🚶 **Roaming** = moving between APs without reconnecting
    - 📏 BSAs should overlap **~10–15%** to avoid dropped connectivity

### 🕸️ Mesh (MBSS)

- Used where running Ethernet to every AP is hard.
- APs use **2 radios**: one for clients (BSS), one for **backhaul** between APs.
- **RAP** = Root AP (wired connection to the network) · **MAP** = Mesh AP
- A protocol picks the best path (like dynamic routing).

### 🔗 Distribution System (DS) & VLANs

- The upstream wired network = **DS (Distribution System)**; AP translates between wireless and wired.
- Each SSID/WLAN ↔ **one VLAN** (e.g., "Jeremy's Wi-Fi" → VLAN 10, "Guest Wi-Fi" → VLAN 11).
- One AP can host multiple SSIDs, connected via a **trunk** carrying those VLANs.
- Each SSID gets a unique BSSID (usually last digit incremented: ...3456, ...3457).

---

## 🔧 5. Other AP Modes

| Mode | Purpose | Notes |
| --- | --- | --- |
| **Repeater** | Extends BSS range by retransmitting | Single radio = same channel → **~50% throughput loss**. Dual radio fixes it (receive on one channel, send on another) |
| **Workgroup Bridge (WGB)** | AP acts as **wireless client** to connect wired devices to Wi-Fi | **uWGB** = 802.11 standard, bridges **one** device. **WGB** = Cisco proprietary, bridges **multiple** wired clients |
| **Outdoor Bridge** | Connects distant networks without cable | Directional antennas. **Point-to-point** or **point-to-multipoint** (hub-and-spoke) |

---

## ⌨️ 6. Syntax / Commands Used

> ℹ️ **No CLI commands were used in this lesson.** Day 55 is purely conceptual (theory/overview); configuration of wireless is covered in later videos in this section. Here is the equivalent list of key terms to know instead:
> 

| 🔑 Term | 📝 Description | 🖥️ Command mode |
| --- | --- | --- |
| — | No CLI syntax introduced | N/A (theory only) |

---

## ✅ 7. Quick Memory Cheat Sheet

- 🔢 2.4 GHz non-overlapping: **1, 6, 11**
- 📶 5 GHz: all channels non-overlapping
- 🏷️ BSSID = AP radio MAC · SSID = network name
- 🤝 ESS = same SSID, unique BSSID, different channels
- 🚫 Clients talk **via the AP**, never directly
- 📡 Wi-Fi 4 = n · Wi-Fi 5 = ac · Wi-Fi 6 = ax
- 🛡️ Wireless needs encryption even inside the LAN
- 🚦 Wireless = CSMA/**CA** · Wired = CSMA/**CD**

---

## ❓ 8. Quiz

**Q1.** When using the 2.4 GHz band, which channels should be selected when using multiple APs?

> ✅ **1, 6, and 11** — they don't overlap, so nearby APs avoid interference.
> 

**Q2.** If an enterprise's network is mostly wired, what is the purpose of an AP?

> ✅ **To connect wireless devices to the wired network** (the DS, distribution system).
> 

**Q3.** Which bands are commonly used by wireless LANs? *(select two)*

> ✅ **2.4 GHz** and **5 GHz**
> 

**Q4.** Which statements about an ESS are true? *(select two)*

> ✅ **Each BSS uses a unique BSSID**
> 
> 
> ✅ **Roaming provides seamless connectivity between APs**
> 
> ❌ Not same-SSID-is-wrong: BSSs in an ESS *should* share the same SSID, and adjacent APs should use *non-overlapping* channels.
> 

**Q5.** Which statement is **NOT** true about an AP that provides multiple BSSs?

> ✅ **Each BSS shares the same BSSID** (false). SSIDs *can* match, but BSSIDs must be unique, and best practice is also a unique SSID per BSS.
>