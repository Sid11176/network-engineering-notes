# Ethernet LAN Switching & Address Resolution Protocol (Part 2)

**Date:** 10.24.2026
**Source:** Jeremy's IT Lab — CCNA 200-301
**Layer:** Layer 2 / Data Link
**Main topics:** Ethernet frames, ARP, ARP table, Ping/ICMP, MAC address table, switch forwarding behavior

---

## 1. Video Roadmap

| Timestamp | Topic |
| --- | --- |
| 0:57 | Ethernet frames |
| 7:41 | ARP Request |
| 10:15 | ARP Reply |
| 12:04 | ARP table |
| 13:05 | GNS3 demo |
| 15:31 | Ping demo |
| 18:28 | Wireshark capture |
| 20:25–22:10 | MAC address table + clearing entries |
| 26:00 | Summary |
| 27:06–31:58 | Quiz questions |

---

## 2. Ethernet Frame

- **Ethernet frame** = Layer 2 PDU.
- It encapsulates the Layer 3 packet inside the Ethernet payload.

### Basic Ethernet II Frame Fields

| Field | Purpose |
| --- | --- |
| Preamble / SFD | Synchronization / start of frame |
| Destination MAC | Receiving NIC/interface MAC |
| Source MAC | Sending NIC/interface MAC |
| Type/Length | Usually identifies Layer 3 protocol, e.g. IPv4 `0x0800`, ARP `0x0806` |
| Payload | Encapsulated L3 packet, e.g. IP packet |
| FCS | Frame Check Sequence — error detection trailer |

### Important Frame-Size Facts

- **Minimum Ethernet payload size = 46 bytes**
- **Minimum Ethernet frame size = 64 bytes** (from Destination MAC through FCS)
- If the L3 packet is smaller than 46 bytes, the sender adds **padding bytes**
- Padding is usually **zeroes**
- **FCS is not part of the payload** — it is the trailer

> **Exam trap:** A series of `00000000` bytes at the end of the Ethernet payload is usually **padding**, not FCS and not part of the ping data.
> 

---

## 3. ARP — Address Resolution Protocol

**ARP** maps an IPv4 address to a MAC address on the local network.

### Why ARP Is Needed

A host knows the destination IP address, but to build an Ethernet frame it needs the destination MAC address.

### ARP Request

- Sent when a host needs to learn the MAC address for an IP
- **Destination MAC = broadcast: `FF:FF:FF:FF:FF:FF`**
- Sent to all hosts on the local network
- Contains: "Who has IP `X.X.X.X`? Tell me your MAC."

### ARP Reply

- Sent by the host that owns the target IP
- **Destination MAC = unicast** to the requester
- Contains: "IP `X.X.X.X` is at MAC `YYYY.YYYY.YYYY`."

### ARP Process Summary

1. Host A wants to send to Host B's IP.
2. Host A checks its ARP table.
3. If no entry exists, Host A sends an **ARP Request** broadcast.
4. Host B replies with an **ARP Reply** unicast.
5. Host A stores IP-to-MAC mapping in its **ARP table**.
6. Host A can now build the Ethernet frame.

> **Exam trap:** ARP Request = **broadcast**. ARP Reply = **unicast**.
> 

---

## 4. ARP Table

**ARP table / ARP cache** = IP address → MAC address mapping.

- Used by hosts, routers, and Layer 3 devices
- Dynamic entries age out
- Static entries can be configured manually

### Viewing & Clearing

| Task | Command |
| --- | --- |
| On Cisco devices | `show arp` |
| On Windows PCs | `arp -a` |
| Clear ARP cache on Cisco | `clear arp-cache` |

### PC `arp -a` Output Fields

| Field | Meaning |
| --- | --- |
| Internet Address | IP address |
| Physical Address | MAC address |
| Type | Dynamic / Static |

> **Exam trap:** "Internet Address, Physical Address, Type" is **ARP table output on a PC**, not Cisco switch MAC address table output.
> 

---

## 5. Ping / ICMP

**Ping** tests Layer 3 reachability using ICMP.

### ICMP Messages Used by Ping

| Message | Direction |
| --- | --- |
| ICMP Echo Request | Sent by the pinging device |
| ICMP Echo Reply | Sent by the target |

### Important Behavior

- Ping uses IP addresses
- Before the first ICMP Echo Request can be sent, the host may need to perform ARP
- **First ping may fail/time out** while ARP resolves
- Later pings usually succeed
- Wireshark can show ARP followed by ICMP

### Useful Wireshark Filters

- `arp`
- `icmp`
- `arp || icmp`

---

## 6. MAC Address Table

**MAC address table / CAM table** = MAC address → switch port / VLAN mapping.

- A switch learns MAC addresses by examining the **source MAC** of incoming frames.

### Cisco Command

```
show mac address-table
```

### Output Fields

| Field | Meaning |
| --- | --- |
| VLAN | VLAN associated with the MAC entry |
| MAC Address | Learned MAC address |
| Type | DYNAMIC / STATIC |
| Ports | Switch interface |

> **Exam trap:** Cisco `show mac address-table` output includes **VLAN, MAC Address, Type, Ports**. It does **not** include IP addresses.
> 

### Dynamic MAC Entries

- Learned automatically from source MAC addresses
- Age out after the default aging time — commonly **300 seconds / 5 minutes**
- Can be cleared manually

### Clear MAC Address Table Commands

```
clear mac address-table dynamic
clear mac address-table dynamic interface interface-id
clear mac address-table dynamic vlan vlan-id
clear mac address-table dynamic address mac-address
```

> **Quiz answer command:** `clear mac address-table dynamic interface interface-id`
> 

---

## 7. Switch Forwarding Behavior

| Frame Type | Switch Behavior |
| --- | --- |
| Known unicast | Forward out the specific port |
| Unknown unicast | Flood out all ports except ingress |
| Broadcast | Flood out all ports except ingress |
| Multicast | May flood unless multicast snooping/control is used |

> **Exam trap:** A switch floods **broadcast** and **unknown unicast** frames out all interfaces except the one it received the frame on.
> 

---

## 8. Hands-On Demo Notes

- GNS3 used to build a small switched network
- Ping used to generate traffic
- Wireshark used to inspect ARP and ICMP
- Clear MAC address table to observe relearning/flooding
- First ping may show ARP Request/Reply before ICMP Echo Request/Reply

### Typical Capture Order

1. ARP Request — broadcast
2. ARP Reply — unicast
3. ICMP Echo Request
4. ICMP Echo Reply

---

## 9. Quiz Questions + Answers

**Q1. You send a 36-byte ping and see a long series of `00000000` at the end of the Ethernet payload. What are these zeroes?**

- **Answer:** b) They are padding bytes.
- **Why:** Ethernet requires a minimum payload size of 46 bytes. If the Layer 3 packet is too small, padding zeroes are added. FCS is a trailer, not payload.

**Q2. Which message is sent to all hosts on the local network?**

- **Answer:** a) ARP request
- **Why:** ARP Requests use the broadcast MAC address `FF:FF:FF:FF:FF:FF`. ARP Replies and ICMP messages are normally unicast.

**Q3. Which fields are present in `show mac address-table` output on a Cisco switch?**

![image.png](Week_1_Pictures/Day4(1)_Quiz3.png)

- **Answer:** c) VLAN, MAC Address, Type, Ports
- **Why:** Cisco switch MAC address table output includes VLAN, MAC, Type, and Ports. "Internet Address, Physical Address, Type" is ARP table output on a PC.

**Q4. Which types of frames does a switch send out of all interfaces except the one it received the frame on?**

- **Answer:** a) Broadcast, unknown unicast
- **Why:** Switches flood broadcast and unknown unicast frames. Known unicast frames are forwarded only out the correct port.

**Q5. Which command clears all dynamic MAC addresses on a specific interface from the MAC address table?**

- **Answer:** d) `clear mac address-table dynamic interface interface-id`
- **Why:** This clears dynamic MAC entries learned on the specified interface. Static entries are not removed by this command.

---

## 10. Key Terms Glossary

| Term | Meaning |
| --- | --- |
| Ethernet frame | Layer 2 PDU |
| FCS | Frame Check Sequence — error-detection trailer |
| Padding | Zero bytes added to meet minimum Ethernet payload size |
| ARP | Maps IPv4 address to MAC address |
| ARP Request | Broadcast message asking "Who has this IP?" |
| ARP Reply | Unicast message giving the MAC for an IP |
| ARP table | IP-to-MAC mapping table |
| ICMP Echo Request | Ping request |
| ICMP Echo Reply | Ping reply |
| Unknown unicast | Frame with destination MAC not yet in MAC table |
| MAC address table | MAC-to-port/VLAN mapping table on a switch |
| Flooding | Sending frame out all ports except ingress |

---

## 11. Exam Traps / Quick Review

- Ethernet minimum payload = **46 bytes**
- Minimum Ethernet frame = **64 bytes**
- Padding zeroes are **not FCS**
- ARP Request = **broadcast**
- ARP Reply = **unicast**
- ARP table maps **IP → MAC**
- MAC address table maps **MAC → port/VLAN**
- Switch floods **broadcast + unknown unicast**
- Known unicast is forwarded out one port
- Cisco MAC table fields: **VLAN, MAC Address, Type, Ports**
- PC ARP table fields: **Internet Address, Physical Address, Type**
- Clear dynamic MAC on interface: `clear mac address-table dynamic interface interface-id`

---

## 12. Commands to Memorize

**MAC address table:**

```
show mac address-table
show mac address-table dynamic
show mac address-table interface interface-id
```

**Clearing MAC entries:**

```
clear mac address-table dynamic
clear mac address-table dynamic interface interface-id
clear mac address-table dynamic vlan vlan-id
clear mac address-table dynamic address mac-address
```

**ARP:**

```
show arp
clear arp-cache
arp -a                      (Windows PC)
```

**Ping:**

```
ping <ip-address>
```

**Wireshark filters:**

```
arp
icmp
arp || icmp
```

---

## Acronyms Reference

| Acronym | Meaning |
| --- | --- |
| **ARP** | Address Resolution Protocol |
| **CAM** | Content-Addressable Memory (CAM table = MAC address table) |
| **CCNA** | Cisco Certified Network Associate |
| **FCS** | Frame Check Sequence |
| **GNS3** | Graphical Network Simulator-3 |
| **ICMP** | Internet Control Message Protocol |
| **IP** | Internet Protocol |
| **IPv4** | Internet Protocol version 4 |
| **L2 / L3** | Layer 2 (Data Link) / Layer 3 (Network) |
| **MAC** | Media Access Control (address) |
| **NIC** | Network Interface Card |
| **PC** | Personal Computer |
| **PDU** | Protocol Data Unit |
| **SFD** | Start Frame Delimiter |
| **VLAN** | Virtual Local Area Network |