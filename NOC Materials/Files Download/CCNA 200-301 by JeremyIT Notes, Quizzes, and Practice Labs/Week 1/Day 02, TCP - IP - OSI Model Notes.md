# 📘 TCP/IP & OSI Models — Study Notes

> 💡 **Big idea:** Encapsulation wraps data as it moves down the sender’s protocol stack. Decapsulation unwraps it as it moves up the receiver’s stack. Routers generally process up to Layer 3; they do not inspect TCP/UDP ports.

---

## 1. 🧱 Common 5-Layer TCP/IP Model

| Layer | Main job | Examples | PDU / unit | Key note |
|---|---|---|---|---|
| **5. Application** | User-facing network services and protocols | HTTP/HTTPS, DNS, SMTP, FTP | Data / message | The interface between applications and the network |
| **4. Transport** | End-to-end, process-to-process delivery | TCP, UDP | Segment (TCP) / Datagram (UDP) | Not hop-by-hop; routers do not process TCP/UDP ports |
| **3. Network / Internet** | Logical addressing and routing | IPv4, IPv6, ICMP | Packet | Routers forward packets based on IP addresses |
| **2. Data Link / Local Area** | Local link delivery, MAC addressing, framing | Ethernet, Wi-Fi, PPP | Frame | Usually has a header + trailer (e.g., FCS for error detection) |
| **1. Physical** | Transmits raw bits/signals over media | Fiber, UTP, radio | Bits | No header/trailer at this layer |

> ⚠️ **Terminology correction:** Your “Internet Layer” is commonly called the **Network Layer**. Your “Local Area Layer” is commonly called the **Data Link Layer** or **Network Access Layer**.

> 💡 **Layer support:** Lower layers provide services to the layers above them. For example, the Transport Layer relies on the Network Layer to move packets across networks.

---

## 2. 📦 Encapsulation Flow — Sender Side

Encapsulation happens on the sending host before transmission.

```text
Application data
↓ + Transport header
Segment (TCP) / Datagram (UDP)
↓ + Network/IP header
Packet
↓ + Data Link header + trailer
Frame
↓
Physical bits / signals
```

### Example: Downloading a file over HTTPS

1. **Application:** HTTP/HTTPS request is created.
2. **Transport:** TCP adds a header → TCP segment.
3. **Network:** IP adds a header → IP packet.
4. **Data Link:** Ethernet/Wi-Fi adds header + trailer → frame.
5. **Physical:** Frame is transmitted as bits/signals.

---

## 3. 📤 Decapsulation — Receiver Side

Decapsulation is the reverse process on the receiving host:

```text
Physical bits/signals
↓
Frame
↓ remove Data Link header/trailer
Packet
↓ remove Network/IP header
Segment / Datagram
↓ remove Transport header
Application data
```

**Layer labels shown in source:** Application, Transport.

> 💡 At the physical layer, bits arrive serially at a given bits-per-second (bps) rate. Decapsulation is not a simple “left-to-right bit read”; the receiver reassembles bits into a frame, then parses structured headers layer by layer.

> 🔁 **Routers:** A router usually removes the incoming Layer 2 frame, inspects the Layer 3 IP packet, then re-encapsulates it in a new Layer 2 frame for the next hop. It does not normally decapsulate up to Layer 4.

---

## 4. 🏷️ PDU Names Cheat Sheet

| Layer | PDU name |
|---|---|
| Application | Data / Message |
| Transport | Segment (TCP) / Datagram (UDP) |
| Network | Packet |
| Data Link | Frame |
| Physical | Bits |

---

## 5. 🆚 TCP/IP vs. OSI

> ⚠️ **Correction:** The OSI model is a **7-layer reference model created by ISO**. It is not a “revised 5-layer model.” The TCP/IP model is the practical protocol suite used on the Internet.

| OSI Layer | Role | Maps roughly to TCP/IP |
|---|---|---|
| **7. Application** | Network services for user applications | Application |
| **6. Presentation** | Data format, encoding, compression, encryption | Application |
| **5. Session** | Starts, manages, and ends sessions/dialogues | Application |
| **4. Transport** | End-to-end delivery, reliability, flow control | Transport |
| **3. Network** | Logical addressing and routing | Network / Internet |
| **2. Data Link** | Framing, MAC addressing, local-link delivery | Data Link |
| **1. Physical** | Bits and signals over physical media | Physical |

> 💡 **Why OSI matters:** It gives a vendor-neutral reference model so devices and software from different companies can interoperate.

---

## 6. 🛠️ Common Pitfalls / Corrections

- ❌ “Transport Layer is hop-to-hop.”  
  ✅ Transport is **end-to-end** (process-to-process). **Data Link** is hop-by-hop.

- ❌ “ISO revised the 5 layers.”  
  ✅ ISO created the **7-layer OSI reference model**. TCP/IP is a separate practical model/suite.

- ❌ “Encapsulation happens during delivery.”  
  ✅ Encapsulation happens on the **sender before transmission**; decapsulation happens on the **receiver**.

- ❌ “Frame is just a header.”  
  ✅ A frame typically has a **header and a trailer** (e.g., FCS).

- ❌ “Decapsulation reads bits left-to-right.”  
  ✅ Bits arrive serially at the physical layer, then the receiver parses structured headers layer by layer.