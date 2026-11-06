# 🌐 TCP & UDP: Layer 4 Protocols (CCNA 1.5)

This note covers the fundamentals of the Transport Layer (Layer 4), focusing on the two main protocols: TCP and UDP. The key objective is to **compare TCP to UDP**.

---

### 🔹 Layer 4 Basics

Layer 4 is responsible for providing **transparent transfer of data** between end hosts. It encapsulates data and uses lower layers to deliver it, without the hosts needing to know the network details.

#### Key Functions

- **Provides (or doesn't provide) services to applications:** These include reliable data transfer, error recovery, sequencing, and flow control.
- **Layer 4 Addressing (Port Numbers):**
    - **Identify Application Protocol:** E.g., Port 80 = HTTP, Port 21 = FTP.
    - **Session Multiplexing:** Allows a host to manage multiple communication sessions simultaneously (e.g., multiple browser tabs).

#### Port Number Ranges (IANA)

| Range | Name | Description |
| --- | --- | --- |
| **0 - 1023** | 🎯 Well-Known Ports | Strictly regulated for major protocols (HTTP, FTP, etc.). |
| **1024 - 49151** | 📝 Registered Ports | Registration required, but less strict. |
| **49152 - 65535** | ⚡ Ephemeral Ports | Used by hosts for randomly selected source ports. Also called private/dynamic ports. |

---

### ✅ TCP (Transmission Control Protocol)

TCP is a feature-rich, **connection-oriented** protocol that provides reliable communication.

#### Key Characteristics

- **Connection-Oriented:** Establishes a connection before sending data.
- **Reliable:** Uses acknowledgments to confirm data receipt. Lost segments are retransmitted.
- **Sequenced:** Uses sequence numbers to reorder segments that arrive out of order.
- **Flow Control:** The destination can tell the source to speed up or slow down data transmission to prevent being overwhelmed.

#### The TCP Header (Key Fields)

- **Source/Destination Port:** (16 bits each) Identifies the application and session.
- **Sequence Number:** Used for sequencing and reliability.
- **Acknowledgment Number:** Used for forward acknowledgment (indicates the next expected sequence number).
- **Flags (SYN, ACK, FIN):** Used to establish and terminate connections.
- **Window Size:** Used for flow control.

#### Connection Establishment: The Three-Way Handshake 🤝

1. **SYN:** PC1 sends a segment with the SYN flag set to initiate a connection.
2. **SYN-ACK:** SRV1 responds with both SYN and ACK flags set.
3. **ACK:** PC1 sends an ACK to confirm, and the connection is established.

#### Connection Termination: The Four-Way Handshake 👋

1. **FIN:** PC1 sends a segment with the FIN flag to request termination.
2. **ACK:** SRV1 acknowledges the FIN.
3. **FIN:** SRV1 sends its own FIN to terminate its side.
4. **ACK:** PC1 acknowledges SRV1's FIN. Connection is closed.

---

### ⚡ UDP (User Datagram Protocol)

UDP is a simple, **connectionless** protocol that provides "best-effort" delivery.

#### Key Characteristics

- **Connectionless:** No connection is established before sending data. It just sends.
- **Unreliable:** No acknowledgments. If a segment is lost, it is not retransmitted.
- **Unsequenced:** No sequence numbers. If segments arrive out of order, UDP has no way to reorder them.
- **No Flow Control:** It sends data as fast as it can.

#### The UDP Header

- **Simple & Small:** Only 4 fields.
- **Fields:** Source Port, Destination Port, Length, Checksum.
- **Less Overhead:** The smaller header and lack of features make UDP faster and more efficient than TCP.

---

### ⚖️ TCP vs. UDP Comparison

| Feature | ✅ TCP | ⚡ UDP |
| --- | --- | --- |
| **Connection** | Connection-Oriented | Connectionless |
| **Reliability** | Reliable (Acks & Retransmissions) | Unreliable (Best-Effort) |
| **Sequencing** | Yes | No |
| **Flow Control** | Yes (Window Size) | No |
| **Overhead** | High (20+ byte header) | Low (8-byte header) |
| **Speed** | Slower | Faster |
| **Use Cases** | File downloads, web browsing, email | Real-time voice/video (VoIP, Zoom), DNS, DHCP |

> **💡 Key Takeaway:** Both TCP and UDP provide **Layer 4 addressing** using port numbers. However, only TCP provides the services of reliable data transfer, error recovery, sequencing, and flow control.
> 

---

### 🔑 Important Well-Known Port Numbers

#### TCP Ports

- **FTP:** 20, 21 (File Transfer Protocol)
- **SSH:** 22 (Secure Shell)
- **Telnet:** 23
- **SMTP:** 25 (Simple Mail Transfer Protocol)
- **HTTP:** 80 (Hypertext Transfer Protocol)
- **POP3:** 110 (Post Office Protocol v3)
- **HTTPS:** 443 (Hypertext Transfer Protocol Secure)

#### UDP Ports

- **DHCP:** 67, 68 (Dynamic Host Configuration Protocol)
- **TFTP:** 69 (Trivial File Transfer Protocol)
- **SNMP:** 161, 162 (Simple Network Management Protocol)
- **Syslog:** 514

#### Both TCP & UDP

- **DNS:** 53 (Domain Name System)