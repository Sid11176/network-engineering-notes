# 📡 Network Devices (10.20.2026)

## 🌐 What is a Network?

- **Computer Network:** A digital telecommunications network that allows nodes to share resources.
- **LAN (Local Area Network):** A network confined to a small geographic area (like an office or home).

## 🖥️ Endpoints (End Hosts)

End hosts (or endpoints) are devices at the edge of the network. They can be **Clients** or **Servers**.

- **Client:** A device that accesses a service made available by a server (e.g., your laptop browsing a website).
- **Server:** A device that provides functions or services for clients (e.g., a web server hosting a website).
- **Key CCNA Concept:** A device’s role is not permanent; it depends on the transaction.
- **Example:** If your PC sends a file to a printer, your PC is the client, and the printer is the server. If someone downloads a file from your PC, your PC is acting as the server.

## 🔀 Network Infrastructure (Switches & Routers)

These devices connect hosts and networks together.

### 🔌 Switch

- Provides connectivity to hosts within the same LAN.
- Has many network interfaces (ports) for end hosts to connect to.
- **Layer:** Operates at **Layer 2 (Data Link)**.

### 📡 Router

- Connects different networks together (e.g., connecting your LAN to the Internet).
- Has fewer network interfaces than a switch.
- **Layer:** Operates at **Layer 3 (Network)**.

## 🛡️ Network Security (Firewalls)

A **Firewall** monitors and controls network traffic based on configured rules.

- **Host-based Firewall:** Software applications installed on a specific machine (like Windows Defender) that filter traffic entering and exiting that host.
- **Network Firewall:** A hardware device placed at the edge of a network to protect the whole network.
- **Next-Generation Firewall (NGFW):** A firewall that combines traditional filtering with more advanced functionalities (e.g., deep packet inspection, intrusion prevention, application awareness).

## 🧾 Cisco Hardware Cheat Sheet

- **Cisco ASA:** Firewall
- **Cisco Firepower:** Firewall (Next-Gen)
- **Cisco ISR:** Router (Integrated Services Router)

## 📝 Practice Quiz & Answers

1. Your company wants to purchase network hardware to which they can plug the 30 PCs in your department. Which is appropriate?
    1. A router
    2. A firewall
    3. A switch
    4. A server
    
    **Answer: c) A switch** — Switches are for connecting many end hosts within the same LAN.
    
2. You received a video file from your friend’s Apple iPhone using AirDrop. What was his iPhone functioning as?
    1. A server
    2. A client
    3. A local area network
    
    **Answer: a) A server** — His phone was providing the file to you, making it the server in that specific
    transaction.
    
3. What is your computer or smartphone functioning as while you watch a YouTube video?
    1. A server
    2. An end host
    3. A client
    
    **Answer: c) A client** — You are accessing a service from YouTube’s servers.
    *Note: It is also technically an “end host,” but “client” is the most specific answer for this role.*
    
4. Your company wants to purchase hardware to connect its separate networks together. What kind of device is appropriate?
    1. A firewall
    2. A host
    3. A LAN
    4. A router
    
    **Answer: d) A router** — Routers route traffic between different networks.
    
5. Your company wants to upgrade its old network firewall to one that provides more advanced functions. What kind of firewall should they purchase?
    1. A host-based firewall
    2. A next-level firewall
    3. A next-generation firewall
    4. A top-layer firewall
    
    **Answer: c) A next-generation firewall** — NGFWs include advanced features beyond basic traffic filtering.