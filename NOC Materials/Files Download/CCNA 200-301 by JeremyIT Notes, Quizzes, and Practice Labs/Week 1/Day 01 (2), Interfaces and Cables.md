# 🔌 Interfaces and Cables (10.20.2026)

## 🧠 Core Concepts

**Bits and Bytes:**
Data is transmitted as **bits** (1s and 0s).  
**8 bits = 1 Byte** (also called an **octet** in networking).

**Ethernet:**
The standard family of technologies used for LANs.

**RJ-45 Connector:**
The standard connector used for copper Ethernet cables (looks like a large telephone jack).

## 🔗 Copper Cables (UTP)

**UTP (Unshielded Twisted Pair):**
The most common and cost-effective network cable. It uses copper wires twisted together to reduce electromagnetic interference.

### Ethernet Standards over UTP

- **10BASE-T:** 10 Mbps over UTP (Category 5e or higher).
- **100BASE-T:** 100 Mbps (Fast Ethernet).
- **1000BASE-T:** 1 Gbps (Gigabit Ethernet).
- **10GBASE-T:** 10 Gbps. **Note:** Strictly requires Cat 6a or higher to avoid interference.

### Pinouts: Straight-Through vs. Crossover

Historically, copper cables required specific pin connections based on the devices you were connecting (Transmit pins must connect to Receive pins).

- **Straight-Through Cable:** Connects different device types (e.g., PC to Switch, Router to Switch).
- **Crossover Cable:** Connects the same device types (e.g., Switch to Switch, Router to Router, PC to PC).

## 🌟 Auto MDI-X

**What it is:**
A modern feature built into network interfaces (switches, routers, etc.).

**How it works:**
It automatically detects which pins are transmitting and which are receiving, and electronically swaps them internally if needed.

**Why it matters:**
- You no longer need crossover cables.
- A straight-through cable will work between any two devices if Auto MDI-X is enabled.

## 💡 Fiber-Optic Cables

Fiber uses light (pulses of light) to transmit data instead of electricity. It is immune to **electromagnetic interference (EMI)** and can travel much further than copper.

### Multimode Fiber

- Uses cheaper light sources (LEDs / VCSELs).
- Core is wider, allowing multiple modes (paths) of light to bounce through.
- **Distance:** Shorter distances (up to ~500m - 2km depending on speed).
- **Cost:** Cheaper than single-mode.

### Single-Mode Fiber

- Uses expensive, precise light sources (lasers).
- Core is very narrow, allowing only a single mode of light to travel straight down the middle.
- **Distance:** Extremely long distances (up to 10km+).
- **Cost:** More expensive.

## 🥊 UTP vs. Fiber Comparison

- **Cost:** UTP is much cheaper.
- **Distance:** Fiber wins (UTP maxes out at 100m).
- **Interference:** Fiber is immune to EMI; UTP is vulnerable but twisting helps.
- **Speed:** Both can handle high speeds, but fiber handles them over much longer distances.

## 🧪 Practice Quiz & Answers

1. You connect two old routers together with a UTP cable, however data is not successfully sent and received between them. What could be the problem?

   a) They are connected with a straight-through cable.  
   b) They are connected with a crossover cable.  
   c) They are operating in Auto MDI-X mode.  

   **Answer: a) They are connected with a straight-through cable.**  
   **Why:** "Old" routers do not have Auto MDI-X. Because they are the same device type, they require a **crossover** cable to connect the TX pins to the RX pins.

2. Your company wants to connect switches in two separate buildings that are about 150 meters apart. They want to keep costs down, if possible. What kind of cable should they use?

   a) UTP  
   b) Single-mode fiber  
   c) Multimode fiber  

   **Answer: c) Multimode fiber.**  
   **Why:** UTP maxes out at 100 meters, so it won't reach 150m. Between the two fiber options, **multimode** is cheaper and perfectly capable of handling 150 meters.

3. Your company wants to connect two offices that are about 3 kilometers apart. They want to keep costs down if possible. Which kind of cable should they use?

   a) UTP  
   b) Single-mode fiber  
   c) Multimode fiber  

   **Answer: b) Single-mode fiber.**  
   **Why:** Multimode is generally used for shorter distances (up to ~500m - 2km). For a 3km distance, **single-mode** fiber is required because its narrow core prevents signal scattering over long distances.

4. A switch has Auto MDI-X enabled on its ports. What would happen if you connect it to an identical switch with a straight-through cable?

   a) They would operate normally.  
   b) They would operate at a reduced speed.  
   c) They would be unable to communicate.  

   **Answer: a) They would operate normally.**  
   **Why:** Thanks to **Auto MDI-X**, the switch automatically detects the connection and adjusts its internal pin logic. A straight-through cable will work fine, even between two identical switches.

5. Your company needs to connect many end hosts to a switch which is in a wiring cabinet on the same office floor as the hosts. What kind of cable should they use?

   a) UTP  
   b) Single-mode fiber  
   c) Multimode fiber  

   **Answer: a) UTP.**  
   **Why:** The distance is under 100 meters (same floor), and UTP is the most cost-effective solution for connecting end hosts like PCs to a local switch.