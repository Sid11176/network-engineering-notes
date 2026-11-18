# 🧭 CCNA Day 26 — OSPF Part 1: Fundamentals & Basic Configuration

> **Course:** Jeremy’s IT Lab — CCNA 200-301
> 
> 
> **Topic:** OSPFv2 fundamentals, areas, and basic Cisco IOS
> configuration
> 
> **Exam objective:** 3.4 — Configure and verify single-area OSPFv2
> 
> **Focus:** Neighbor adjacencies, point-to-point, broadcast, and router
> ID
> 
> **Scope of this lesson:** OSPF overview, area concepts, and
> introductory configuration. Neighbor formation details are developed
> further in the next lesson.
> 

---

## 🎯 1. OSPF at a glance

**OSPF (Open Shortest Path First)** is a **link-state dynamic routing
protocol**. It uses the **Shortest Path First (SPF) algorithm**, also
known as **Dijkstra’s algorithm**, to calculate the best routes.

| OSPF version | Main use |
| --- | --- |
| OSPFv1 | Older version; no longer in use |
| **OSPFv2** | Typically used for IPv4; the
version covered by the CCNA OSPF
objective |
| OSPFv3 | Developed for IPv6; can also
support IPv4 |

### 🔁 Link-state vs. distance-vector

- **Distance-vector:** Often described as “routing by rumor.” Routers
share route information and metrics with neighbors, but do not build
a complete network map.
- **Link-state (OSPF):** Routers advertise information about their
links. The information is flooded through the area so routers build
the same network map, then each router independently calculates its
best routes.

**Key trade-off:** Link-state protocols use more router resources
because they share and process more information, but they generally
react faster to network changes than distance-vector protocols.

---

## 🗺️ 2. OSPF operation: LSA, LSDB, and SPF

### Important terms

| Term | Meaning |
| --- | --- |
| **LSA — Link-State
Advertisement** | Information a router advertises
about its links and network
topology |
| **LSDB — Link-State Database** | The database containing the LSAs;
represents the network map |
| **Flooding** | Sending LSAs to OSPF neighbors so
the information spreads through the
area |
| **SPF algorithm** | Dijkstra’s algorithm, used to
calculate the best paths |
| **Router ID (RID)** | Identifier for an OSPF router |

### How the process works

1. **Become neighbors:** Routers on the same segment establish OSPF
neighbor relationships.
2. **Exchange and flood LSAs:** Routers share link-state information.
LSAs are flooded through the area until routers have the same LSDB.
3. **Calculate routes:** Each router independently runs SPF against its
LSDB and installs its best routes in its routing table.

### What an LSA can describe

The lesson’s example shows an LSA containing information such as:

- The advertising router’s **RID**.
- The network connected to the relevant interface.
- The interface’s OSPF **cost**.

In the example, R4 advertises a newly enabled `G1/0` network. Its LSA is
flooded through the area, added to the routers’ LSDBs, and each router
recalculates its best route to that network.

> ⏱️ **LSA aging (as stated in the lesson):** An individual LSA has a
default aging timer of **30 minutes**. It is flooded again after the
timer expires.
> 

**Remember:** Routers in the same OSPF area maintain the same LSDB. They
do not necessarily have identical routing tables, because each router
calculates routes from its own position in the topology.

---

## 🧩 3. OSPF areas

An **area** is a set of routers and links that share the same LSDB.

### Why use areas?

A small network can operate as a single area. In a large network, one
enormous area can create disadvantages:

- SPF calculations take longer.
- Routers require more processing power.
- A large LSDB consumes more memory.
- A topology change can cause LSAs to be flooded widely, prompting
many routers to recalculate routes.

Dividing a large OSPF network into smaller areas helps limit these
effects.

### Backbone area — Area 0

- **Area 0** is the OSPF **backbone area**.
- All other OSPF areas must connect to Area 0.
- A non-backbone area should have an Area Border Router (ABR)
connecting it to Area 0.
- For single-area OSPF, Area 0 is recommended as best practice, but it
is **not mandatory**.

### OSPF area terminology

| Term | Definition |
| --- | --- |
| **Internal router** | A router with all OSPF-enabled
interfaces in the same area |
| **ABR — Area Border Router** | A router with interfaces in
multiple OSPF areas; it maintains a
separate LSDB for each area |
| **Backbone router** | A router with at least one
interface in Area 0; an ABR
connected to Area 0 is also a
backbone router |
| **Intra-area route** | A route to a destination within the
same OSPF area |
| **Interarea route** | A route to a destination in a
different OSPF area |
| **ASBR — Autonomous System
Boundary Router** | An OSPF router that connects the
OSPF domain to an external network
and introduces external routing
information |

**ABR design note from the lesson:** It recommends connecting an ABR to
a maximum of two areas; connecting it to three or more may overburden
the router.

### Three OSPF area design rules

1. **Areas should be contiguous.** An area should form one connected
section, not be split into disconnected pieces.
2. **Every non-backbone area must connect to Area 0 through an ABR.**
An area connected only to another non-backbone area does not satisfy
this rule.
3. **Interfaces in the same subnet must be in the same OSPF area.** If
interfaces sharing a subnet are assigned to different areas, those
routers will not become OSPF neighbors over that segment or exchange
information there.

> 🧠 **Exam distinction:** An ABR connects OSPF *areas*. An ASBR
connects the OSPF domain to an *external network*. They are not
interchangeable terms.
> 

---

## ⚙️ 4. Basic single-area OSPFv2 configuration

The lesson uses Cisco IOS commands and places all interfaces in **Area
0** for its single-area example.

### Step 1 — Start an OSPF process

```
Router(config)# router ospf 1
```

- Enters OSPF router configuration mode.
- `1` is the **OSPF process ID**.
- A router can run multiple OSPF processes.
- The process ID is **locally significant**: neighboring routers do
not need to use the same process ID.
- The process ID is separate from the OSPF area number.

### Step 2 — Activate OSPF on interfaces with `network`

```
Router(config-router)# network 10.0.12.0 0.0.0.15 area 0
```

General syntax:

```
network <network-address> <wildcard-mask> area <area-id>
```

- OSPF uses a **wildcard mask**, the inverse of the subnet mask.
- The command matches local interface IP addresses in the specified
range and activates OSPF on those interfaces in the specified area.
- It does **not** simply mean “advertise this network.” Its purpose
here is to select the interfaces on which OSPF will run.
- The area must be included in the command.

**Example from the lesson:** An interface with IP `10.0.12.1` falls
within `10.0.12.0/28`, whose wildcard mask is `0.0.0.15`. OSPF is
therefore activated on that interface in Area 0.

### Step 3 — Make an interface passive

```
Router(config-router)# passive-interface g2/0
```

- Stops OSPF Hello messages from being sent out of that interface.
- The router does not attempt to discover OSPF neighbors through that
interface.
- The connected subnet can still be included in OSPF advertisements to
other OSPF neighbors.
- Use this on interfaces that do not have OSPF neighbors, according to
the lesson.

### Step 4 — Advertise a default route into OSPF

First, configure a default route on the router that has the external
connection:

```
Router(config)# ip route 0.0.0.0 0.0.0.0 <next-hop-ip>
```

Then, under OSPF router configuration mode:

```
Router(config-router)# default-information originate
```

- Advertises the router’s existing default route into OSPF.
- OSPF creates and floods an LSA for the default route.
- Other OSPF routers can then install a default route pointing toward
this router.
- In the lesson’s example, the router connected to the Internet
becomes an **ASBR**.

---

## 🆔 5. OSPF Router ID

The Router ID identifies the OSPF router. The lesson gives this
selection order:

1. **Manually configured router ID** — takes priority.
2. If not manually configured, the **highest IP address on a loopback
interface**.
3. If there is no loopback interface, the **highest IP address on a
physical interface**.

### Manually configure the Router ID

From OSPF router configuration mode:

```
Router(config-router)# router-id 1.1.1.1
```

If OSPF is already running, the new ID may not take effect immediately.
The lesson shows two ways to apply it:

- Reload the router; or
- Reset the OSPF process from privileged EXEC mode:

```
Router# clear ip ospf process
```

The router asks for confirmation. In the demonstrated prompt, `[no]`
means **No is the default** if you simply press Enter.

> ⚠️ **Operational caution:** Clearing the OSPF process resets OSPF. The
router can temporarily lose its OSPF-learned routes and forwarding to
those destinations may be interrupted. The lesson treats this as
acceptable in a lab, but warns against doing it casually in a real
network.
> 

---

## 🔍 6. Verification and additional OSPF settings

### Useful command: `show ip protocols`

```
Router# show ip protocols
```

The lesson uses this command to inspect OSPF information, including:

- OSPF process ID.
- Router ID.
- Whether the router is an ASBR.
- Number and types of areas.
- Maximum paths.
- OSPF `network` statements.
- Passive interfaces.
- OSPF neighbors.
- OSPF administrative distance.

### Other settings shown

| Setting | Command / value | Lesson note |
| --- | --- | --- |
| Maximum paths | `maximum-paths 8` | OSPF supports
equal-cost multipath
(ECMP); default shown
is 4 paths |
| Administrative distance | `distance 85` | OSPF’s default AD is
**110**; changing it to
85 makes OSPF routes
preferred over routes
with a higher AD, such
as the example’s EIGRP
routes |
| Default OSPF AD | `110` | Displayed by
`show ip protocols` |

**Load balancing:** The lesson states that OSPF supports equal-cost load
balancing, not unequal-cost load balancing. The default maximum is four
paths.

### Command syntax recap

```
router ospf <process-id>
network <network-address> <wildcard-mask> area <area-id>
passive-interface <interface-id>
default-information originate
router-id <A.B.C.D>
clear ip ospf process
show ip protocols
maximum-paths <number>
distance <administrative-distance>
```

The first, second, third, fourth, fifth, eighth, and ninth commands are
entered in the relevant router/OSPF configuration mode as demonstrated.
`clear ip ospf process` and `show ip protocols` are privileged EXEC
commands.

---

## 📌 7. Key facts to memorize

- OSPF is a **link-state** routing protocol.
- OSPF uses **Dijkstra’s SPF algorithm**.
- OSPFv2 is the version typically used for IPv4 and is the focus of
the CCNA objective.
- LSAs are stored in the LSDB; LSAs are flooded so routers in an area
share the same topology database.
- OSPF’s three broad steps: **neighbor formation → LSA
exchange/flooding → SPF calculation and route installation**.
- Area 0 is the backbone. Other areas must connect to it.
- Single-area OSPF can use an area number other than 0, but Area 0 is
recommended.
- OSPF process IDs are locally significant and do not need to match
between neighbors.
- The OSPF `network` command requires an area and uses a wildcard mask
to select interfaces.
- A passive interface does not send Hellos, but its subnet can still
be advertised through other OSPF neighbors.
- `default-information originate` advertises the router’s default
route into OSPF.
- Router ID selection: **manual → highest loopback IP → highest
physical-interface IP**.
- OSPF default administrative distance: **110**.
- Default maximum equal-cost paths shown: **4**.
- **ABR = between areas; ASBR = between OSPF and external routing
domain.**

---

# 📝 8. Lesson Quiz

Try answering before opening the answer key. Questions and answers below
follow the lesson transcription.

## Question 1 — OSPF statements

**Which two statements are NOT true? Select two.**

- A. In multi-area OSPF networks, all non-backbone areas must have an
ABR connected to Area 0.
- B. Single-area OSPF must use Area 0.
- C. Two OSPF routers with different process IDs can become OSPF
neighbors.
- D. The OSPF area must be specified in the `network` command.
- E. An ASBR connects the internal OSPF network to networks outside
the OSPF domain.
- F. The OSPF process ID must match the area number.
- ✅ Show answer and explanation
    
    **Correct answers: B and F.**
    
    - **B is false:** Area 0 is best practice, but a single-area OSPF
    network can use another area number.
    - **F is false:** The process ID and area number are separate. Process
    IDs are locally significant and do not have to match between
    routers.
    - A, C, D, and E are true according to the lesson.

## Question 2 — Select the `network` command

You want to activate OSPF on both R1 interfaces with one command:

- `G0/1`: `10.0.12.1/28`
- `G0/2`: `10.0.13.1/26`

**Which command activates OSPF on both interfaces in Area 0?**

- ✅ Show answer and explanation
    
    **Correct answer: C**
    
    ```
    network 10.0.12.0 0.0.1.255 area 0
    ```
    
    This wildcard range includes both interface IP addresses, so one command
    matches both interfaces.
    

## Question 3 — Identify router roles

Using the OSPF network diagram from the lesson, answer:

1. How many backbone routers are there?
2. How many ABRs are there?
3. How many ASBRs are there?
- ✅ Show answers
    1. **4 backbone routers** — each has at least one interface in Area
        1. 
    2. **3 ABRs** — each has interfaces in two separate areas.
    3. **1 ASBR** — the router connecting the OSPF network to the
    Internet/external network.

## Question 4 — Make R1 an ASBR

**Which configuration makes R1 an OSPF ASBR in the lesson’s example?**

- ✅ Show answer and explanation
    
    **Correct answer: B — configure a default route, then advertise it
    into OSPF.**
    
    ```
    ip route 0.0.0.0 0.0.0.0 <next-hop-ip>
    router ospf 1
     default-information originate
    ```
    
    - Network statements only activate OSPF on interfaces.
    - A wildcard of `0.0.0.0` matches all IPv4 interface addresses when
    used as the wildcard for `0.0.0.0/0`.
    - Option D in the lesson is not a valid command.

## Question 5 — Manually configure the Router ID

**Which command manually configures the OSPF Router ID?**

- ✅ Show answer and explanation
    
    **Correct answer: A**
    
    ```
    router ospf 1
     router-id 1.1.1.1
    ```
    
    The command is `router-id` in OSPF router configuration mode. The lesson
    distinguishes this from EIGRP’s `eigrp router-id` syntax. If OSPF is
    already running, reset the process or reload for the new ID to take
    effect.
    

---

## ⭐ Bonus — Boson ExSim question

You issue `default-information originate` on RouterA.

**Which two statements are true? Select two.**

- A. OSPF will advertise RouterA’s gateway of last resort.
- B. RouterA will become the OSPF ABR.
- C. OSPF will summarize all of RouterA’s directly connected routes.
- D. RouterA will become the OSPF ASBR.
- E. OSPF will redistribute all of RouterA’s directly connected
routes.
- ✅ Show answer and explanation
    
    **Correct answers: A and D.**
    
    - **A is true:** The command advertises RouterA’s default route
    (gateway of last resort) into OSPF.
    - **D is true:** RouterA acts as an ASBR by introducing the external
    default route into the OSPF domain.
    - **B is false:** An ABR has interfaces in multiple OSPF areas;
    advertising a default route does not make a router an ABR.
    - **C and E are false:** The command does not summarize or
    redistribute all directly connected routes.

---

## 🧪 9. Suggested Packet Tracer practice

Use a small single-area topology to reinforce the lesson:

1. Configure IP addresses on router interfaces.
2. Start OSPF with `router ospf 1`.
3. Use `network` statements with wildcard masks and `area 0`.
4. Make a LAN-facing interface passive if no OSPF neighbor should form
there.
5. Configure a default route on the edge router and advertise it with
`default-information originate`.
6. Set a manual Router ID and observe the effect of
`clear ip ospf process` in the lab.
7. Verify the configuration with `show ip protocols`.

> **Scope reminder:** This lesson is an introduction. Detailed OSPF
neighbor adjacency behavior and network types are covered in
subsequent lessons.
> 

---

## 📚 Source

Organized from the supplied transcription of **Jeremy’s IT Lab — “Free
CCNA | OSPF Part 1 | Day 26 | CCNA 200-301 Complete Course.”**
Explanations and quiz answers are based on the lesson transcription.