# 📡 Ethernet LAN Switching — Part 1

**Date:** 10.23.2026

---

## 1. OSI Model Context

- **Layer 1 (Physical):** Deals with the actual cables, connectors, and signals (electrical pulses, light).
- **Layer 2 (Data Link):** Deals with **Ethernet Frames** and **MAC Addressing**.

> **Note:** The focus here is on the logical addressing (MAC) and how devices talk on the same local network.

---

## 2. The Ethernet Frame Structure

Understanding the frame is crucial for troubleshooting and understanding how switches operate.

| Field | Size | Description |
| :--- | :--- | :--- |
| **Preamble** | 7 Bytes | **Clock Synchronization.** A pattern of alternating 1s and 0s (`10101010`) used so the receiving device can synchronize its clock with the sender. |
| **SFD** (Start Frame Delimiter) | 1 Byte | **Frame Start.** Ends with `10101011`. Tells the receiver "The next bits are the destination MAC address." |
| **Destination MAC** | 6 Bytes | The intended recipient of the frame. |
| **Source MAC** | 6 Bytes | The sender of the frame. |
| **Type / Length** | 2 Bytes | Tells the receiver what protocol is inside (e.g., IPv4, IPv6, ARP). |
| **Data / Payload** | 46 – 1500 Bytes | The actual encapsulated data (PDU from Layer 3). |
| **FCS** (Frame Check Sequence) | 4 Bytes | **Error Detection.** Uses CRC (Cyclic Redundancy Check) to detect corrupted data. |

> 💡 **Exam Tip:** The Preamble + SFD are often considered the "overhead" to prepare the receiver. If the FCS fails, the frame is **dropped**.

---

## 3. MAC Addresses (Media Access Control)

A MAC address is a unique identifier assigned to a network interface card (NIC).

- **Format:** Hexadecimal (Base 16).
- **Length:** **48 bits** (6 bytes).
- **Notation:** Usually written as 12 hex digits (e.g., `E8BA.7011.2874`).

### The Structure of a MAC Address

**1. OUI (Organizationally Unique Identifier)**
- The **first 24 bits** (first half) of the address.
- It identifies the **Manufacturer** (e.g., Cisco, Apple, Intel).
- **Quiz Example:** In `E8BA.7011.2874`, the OUI is **`E8BA.70`**.

**2. Device Identifier**
- The **last 24 bits** (second half).
- Unique to that specific device (assigned by the manufacturer).

### Types of Addresses

- **Unicast:** A single specific device.
- **Broadcast:** All devices on the subnet (`FF:FF:FF:FF:FF:FF`).
- **Multicast:** A group of devices.

---

## 4. How Switches Operate (The MAC Address Table)

A switch is a **Layer 2** device. Its main job is to forward frames based on the **Destination MAC Address**. To do this, it builds a **MAC Address Table** (also called a **CAM table**).

### The Learning Process (Quiz Question)

> **Question:** Which field does a switch use to populate its MAC address table?
>
> **Answer:** The **Source MAC Address**.

**Why?** When a frame enters an interface, the switch looks at the **Source MAC** and thinks: *"Ah, this device is connected to this port. I will add it to my table."*

### Forwarding Logic

**1. Known Unicast**
- The switch looks up the **Destination MAC** in its table.
- If it finds a match, it **forwards** the frame out of the specific interface associated with that MAC.

**2. Unknown Unicast (Flooding)**
- The switch looks up the **Destination MAC**.
- If the MAC is **not** in the table, it **floods** the frame.
- **Flooding:** The switch sends the frame out of **all interfaces except the one it was received on**.

> **Note:** It does not forward out the interface it came from (that would be a loop).

> ⚠️ **Important Nuance:** The switch learns the Source MAC *before* it forwards the frame. So, if PC-A sends a frame to PC-B, and the switch doesn't know where PC-B is, it floods the frame. However, it **does** learn PC-A's location during this process.

---

## 5. Quiz Summary & Answers

A quick review of the quiz questions provided:

| # | Question | Answer |
| :--- | :--- | :--- |
| 1 | Which field of an Ethernet frame provides receiver clock synchronization? | ✅ **Preamble** (Allows the receiver to sync its clock). |
| 2 | How long is the physical address of a network device? | ✅ **48 bits** (Because the MAC address is 6 bytes). |
| 3 | What is the OUI of this MAC address? `E8BA.7011.2874` | ✅ **`E8BA.70`** (The first 24 bits / 3 bytes). |
| 4 | Which field of an Ethernet frame does a switch use to populate its MAC address table? | ✅ **Source MAC Address** (The switch learns which port the sender is on). |
| 5 | What kind of frame does a switch flood out of all interfaces except the one it was received on? | ✅ **Unknown Unicast** (The destination MAC is not yet in the MAC table). |

---

## 📝 Mnemonic / Cheat Sheet

| Term | Mnemonic |
| :--- | :--- |
| **Preamble** | **P**hysical **P**rep (Sync). |
| **Source MAC** | **S**witch **S**aves (Learning). |
| **Destination MAC** | **D**irect **D**elivery (Forwarding). |
| **Unknown** | **U**niversal Flood (Sending everywhere). |

---

## 📊 Comparison: Hub vs. Switch (Context for Part 1)

| Feature | Hub (Layer 1) | Switch (Layer 2) |
| :--- | :--- | :--- |
| **Traffic** | Floods everything | Forwards intelligently |
| **Table** | None | MAC Address Table |
| **Collision Domain** | All ports share one | Each port is its own |

> **Note:** This video focused heavily on Switch logic, which is the foundation for understanding **VLANs** and **Spanning Tree Protocol (STP)** in later videos.