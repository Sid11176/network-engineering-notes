# IP and VLAN Plan

Last updated: Session 5 (`v2-standout.pkt`, complete)

Scope: Sessions 1–5.

- Sessions 1–2 entries are Active in `v0-foundation.pkt` (unchanged).
- Session 3 entries are marked **S3** and apply to `v1-core.pkt`.
- Session 4 entries are marked **S4** and apply to `v1-core.pkt`.
- Session 5 entries are marked **S5** and apply to `v2-standout.pkt`.
- Later-session items (VLAN 99 management, hardening) are intentionally omitted.

---

## 1. VLAN plan

| Site | VLAN | Name | Subnet | Default gateway |
| --- | --- | --- | --- | --- |
| HQ | 10 | ADMIN | 192.168.10.0/24 | 192.168.10.1 |
| HQ | 20 | USERS | 192.168.20.0/24 | 192.168.20.1 |
| HQ | 30 | IT | 192.168.30.0/24 | 192.168.30.1 |
| HQ | 50 | SERVERS (**S5**) | 192.168.50.0/24 | 192.168.50.1 |
| Branch | 1 | LAN BRANCH (default VLAN) | 192.168.40.0/24 | 192.168.40.1 |

Notes:
- No VTP is used. VLANs are created manually on SW1, SW2 and SW3 (**S5**).
- The branch is a single flat LAN, so SW4 uses VLAN 1.
- R1 Gi0/0 carries no IP address; it is only the 802.1Q trunk parent.
- VLAN 50 exists on all three HQ switches and is allowed on every HQ trunk (**S5**).

---

## 2. Device inventory

| Device | Model | Role | Added |
| --- | --- | --- | --- |
| R1 | 2911 | HQ router, router-on-a-stick (VLANs 10, 20, 30, 50), DHCP server (S3), NAT/PAT (S3), primary ISP path, floating backup default route via R2 (S4) | S1 |
| R2 | 2911 | Branch router, branch DHCP server (S4), NAT/PAT for the backup ISP path (S4), route to VLAN 50 via R1 (**S5**) | S2 |
| R3 | 2911 | Simulated ISP router, hosts the simulated public server LAN (**S5**) | S3 |
| SW1 | 2960 | HQ distribution switch, STP root primary (**S5**) | S1 |
| SW2 | 2960 | HQ switch, STP root secondary, HQ PC access (**S5**) | **S5** |
| SW3 | 2960 | HQ switch, server access (**S5**) | **S5** |
| SW4 | 2960 | Branch access switch | S2 |
| PC-ADMIN | PC | VLAN 10 client | S1 |
| PC-USERS | PC | VLAN 20 client | S1 |
| PC-IT | PC | VLAN 30 client | S1 |
| PC-BRANCH | PC | Branch client | S2 |
| DNS Server | Server | Internal DNS, VLAN 50 | **S5** |
| Web Server | Server | Intranet web server, VLAN 50 | **S5** |
| Public Web Server | Server | Simulated public web server behind R3 | **S5** |

---

## 3. Interface and addressing map

### R1 — HQ router

| Interface | Encapsulation | IP address | Purpose | NAT role |
| --- | --- | --- | --- | --- |
| Gi0/0 | — | no IP | Physical trunk to SW1 Gi0/1 | — |
| Gi0/0.10 | dot1Q 10 | 192.168.10.1/24 | VLAN 10 gateway | inside |
| Gi0/0.20 | dot1Q 20 | 192.168.20.1/24 | VLAN 20 gateway | inside |
| Gi0/0.30 | dot1Q 30 | 192.168.30.1/24 | VLAN 30 gateway | inside |
| Gi0/0.50 | dot1Q 50 | **192.168.50.1/24** | VLAN 50 gateway (**S5**) | **inside** (S5) |
| Gi0/1 | — | 10.0.0.1/30 | Transit link to R2 Gi0/0 | neither |
| Gi0/2 | — | 203.0.113.1/30 | ISP link to R3 Gi0/0 | outside |

### R2 — Branch router

| Interface | IP address | Purpose | NAT role |
| --- | --- | --- | --- |
| Gi0/0 | 10.0.0.2/30 | Transit link to R1 Gi0/1 | inside (S4) |
| Gi0/1 | 192.168.40.1/24 | Branch LAN gateway to SW4 | inside (S4) |
| Gi0/2 | 203.0.113.5/30 | Backup ISP link to R3 Gi0/1 | outside (S4) |

### R3 — Simulated ISP router

| Interface | IP address | Purpose |
| --- | --- | --- |
| Gi0/0 | 203.0.113.2/30 | ISP link to R1 Gi0/2 |
| Gi0/1 | 203.0.113.6/30 | Backup ISP link to R2 Gi0/2 |
| Gi0/2 | **198.51.100.1/24** (**S5**) | Public server LAN to Public Web Server |
| Loopback0 | 8.8.8.8/32 | Simulated Internet test destination |

### SW1 — HQ distribution switch

| Interface | Mode | VLAN / allowed VLANs | Purpose |
| --- | --- | --- | --- |
| Gi0/1 | trunk | allowed 10, 20, 30, **50** (S5) | Uplink to R1 Gi0/0 |
| Fa0/23 | trunk | allowed 10, 20, 30, 50 | To SW2 Fa0/23 (**S5**) |
| Fa0/24 | trunk | allowed 10, 20, 30, 50 | To SW3 Fa0/23 (**S5**) |
| Fa0/1–Fa0/22 | access | VLAN 1 | Unused in Session 5 (the HQ PCs moved to SW2) |

### SW2 — HQ switch (**S5**)

| Interface | Mode | VLAN / allowed VLANs | Purpose |
| --- | --- | --- | --- |
| Fa0/23 | trunk | allowed 10, 20, 30, 50 | To SW1 Fa0/23 |
| Fa0/24 | trunk | allowed 10, 20, 30, 50 | To SW3 Fa0/24 |
| Fa0/1 | access | VLAN 10 | PC-ADMIN |
| Fa0/2 | access | VLAN 20 | PC-USERS |
| Fa0/3 | access | VLAN 30 | PC-IT |
| Fa0/4–Fa0/22 | access | VLAN 1 | Unused in Session 5 |

### SW3 — HQ switch (**S5**)

| Interface | Mode | VLAN / allowed VLANs | Purpose |
| --- | --- | --- | --- |
| Fa0/23 | trunk | allowed 10, 20, 30, 50 | To SW1 Fa0/24 |
| Fa0/24 | trunk | allowed 10, 20, 30, 50 | To SW2 Fa0/24 |
| Fa0/1 | access | VLAN 50 | DNS Server |
| Fa0/2 | access | VLAN 50 | Web Server |
| Fa0/3–Fa0/22 | access | VLAN 1 | Unused in Session 5 |

### SW4 — Branch switch

| Interface | Mode | VLAN | Purpose |
| --- | --- | --- | --- |
| Gi0/1 | access | VLAN 1 | Uplink to R2 Gi0/1 |
| Fa0/1 | access | VLAN 1 | PC-BRANCH |
| Fa0/2–Fa0/24 | access | VLAN 1 | Unused in Sessions 1–5 |

---

## 4. Physical link map

| From | Port | To | Port | Cable | Link type | Added |
| --- | --- | --- | --- | --- | --- | --- |
| R1 | Gi0/0 | SW1 | Gi0/1 | straight-through | 802.1Q trunk, allowed VLANs 10, 20, 30, 50 | S1 (VLAN 50 added S5) |
| R1 | Gi0/1 | R2 | Gi0/0 | crossover | Routed /30 | S2 |
| R2 | Gi0/1 | SW4 | Gi0/1 | straight-through | Access, VLAN 1 | S2 |
| SW4 | Fa0/1 | PC-BRANCH | NIC | straight-through | Access, VLAN 1 | S2 |
| R1 | Gi0/2 | R3 | Gi0/0 | crossover | Routed /30 (primary ISP) | S3 |
| R2 | Gi0/2 | R3 | Gi0/1 | crossover | Routed /30 (backup ISP) | S4 |
| SW1 | Fa0/23 | SW2 | Fa0/23 | crossover | 802.1Q trunk, allowed VLANs 10, 20, 30, 50 | **S5** |
| SW1 | Fa0/24 | SW3 | Fa0/23 | crossover | 802.1Q trunk, allowed VLANs 10, 20, 30, 50 | **S5** |
| SW2 | Fa0/24 | SW3 | Fa0/24 | crossover | 802.1Q trunk, allowed VLANs 10, 20, 30, 50 | **S5** |
| SW2 | Fa0/1 | PC-ADMIN | NIC | straight-through | Access, VLAN 10 (moved from SW1 Fa0/1) | **S5** |
| SW2 | Fa0/2 | PC-USERS | NIC | straight-through | Access, VLAN 20 (moved from SW1 Fa0/2) | **S5** |
| SW2 | Fa0/3 | PC-IT | NIC | straight-through | Access, VLAN 30 (moved from SW1 Fa0/3) | **S5** |
| SW3 | Fa0/1 | DNS Server | NIC | straight-through | Access, VLAN 50 | **S5** |
| SW3 | Fa0/2 | Web Server | NIC | straight-through | Access, VLAN 50 | **S5** |
| R3 | Gi0/2 | Public Web Server | NIC | crossover | Routed /24 (public server LAN) | **S5** |

---

## 5. Routed links

| Link | Subnet | Endpoint A | Endpoint B | Added |
| --- | --- | --- | --- | --- |
| R1–R2 transit | 10.0.0.0/30 | R1 Gi0/1: 10.0.0.1 | R2 Gi0/0: 10.0.0.2 | S2 |
| Branch LAN | 192.168.40.0/24 | R2 Gi0/1: 192.168.40.1 | SW4 / PC-BRANCH | S2 |
| R1–R3 ISP link (primary) | 203.0.113.0/30 | R1 Gi0/2: 203.0.113.1 | R3 Gi0/0: 203.0.113.2 | S3 |
| R2–R3 ISP link (backup) | 203.0.113.4/30 | R2 Gi0/2: 203.0.113.5 | R3 Gi0/1: 203.0.113.6 | S4 |
| R3 test loopback | 8.8.8.8/32 | R3 Loopback0 | — | S3 |
| R3 public server LAN | 198.51.100.0/24 | R3 Gi0/2: 198.51.100.1 | Public Web Server: 198.51.100.10 | **S5** |

`203.0.113.0/24` and `198.51.100.0/24` are documentation-only ranges, and
`8.8.8.8/32` is a simulated loopback. A ping to any of them proves reachability
to that simulated destination only, not real Internet access.

---

## 6. Static and default routing

| Router | Destination | Next hop | AD | Purpose | Added |
| --- | --- | --- | --- | --- | --- |
| R1 | 192.168.40.0/24 | 10.0.0.2 | 1 | Route to branch LAN | S2 |
| R2 | 192.168.10.0/24 | 10.0.0.1 | 1 | Route to HQ VLAN 10 | S2 |
| R2 | 192.168.20.0/24 | 10.0.0.1 | 1 | Route to HQ VLAN 20 | S2 |
| R2 | 192.168.30.0/24 | 10.0.0.1 | 1 | Route to HQ VLAN 30 | S2 |
| R2 | 192.168.50.0/24 | 10.0.0.1 | 1 | Route to HQ VLAN 50 | **S5** |
| R1 | 0.0.0.0/0 | 203.0.113.2 | 1 | Primary default route to R3 | S3 |
| R1 | 0.0.0.0/0 | 10.0.0.2 | 10 | Floating backup default route via R2 | S4 |
| R2 | 0.0.0.0/0 | 203.0.113.6 | 1 | Branch default route to R3 | S4 |

R3 has its connected routes (`203.0.113.0/30`, `203.0.113.4/30`,
`198.51.100.0/24` (**S5**), `8.8.8.8/32`). Because R1 and R2 translate private
client addresses to `203.0.113.1` and `203.0.113.5` respectively, replies are
sent to a directly connected address and R3 needs no route to the private
subnets. This also applies to VLAN 50 traffic once it is in the NAT ACLs.

Failover behavior (verified in `verification/failover.md`):
- Normal: R1 installs the AD 1 default via R3. The AD 10 route is configured
  but not installed.
- R1 Gi0/2 down: R1 installs the AD 10 default via 10.0.0.2. HQ traffic goes
  R1 → R2 → R3 and is translated on R2 to `203.0.113.5`.
- R1 Gi0/2 restored: the AD 1 default returns.

Limitation: the floating route has no IP SLA or tracking. It reacts to a
failed R1–R3 link, not to an ISP outage beyond R3 while Gi0/2 stays up.

---

## 7. DHCP

HQ DHCP is served by R1 (S3). Branch DHCP is served by R2 (S4). The first
column shows the DHCP pool names. VLAN 50 has no DHCP pool because the servers
use static addresses (section 11).

| Pool name | Network | Default router | DNS server | Excluded addresses | Dynamic range | Server |
| --- | --- | --- | --- | --- | --- | --- |
| VLAN10-ADMIN | 192.168.10.0/24 | 192.168.10.1 | 192.168.50.10 (**S5**) | .1–.50 | .51–.254 | R1 |
| VLAN20-USERS | 192.168.20.0/24 | 192.168.20.1 | 192.168.50.10 (**S5**) | .1–.50 | .51–.254 | R1 |
| VLAN30-IT | 192.168.30.0/24 | 192.168.30.1 | 192.168.50.10 (**S5**) | .1–.50 | .51–.254 | R1 |
| BRANCH-LAN | 192.168.40.0/24 | 192.168.40.1 | 192.168.50.10 (**S5**) | .1–.50 | .51–.254 | R2 |

- The excluded range matches the address reservation in section 10.
- `dns-server 192.168.50.10` was added to all four pools in Session 5.
- Leases observed during Session 5 verification: PC-ADMIN `192.168.10.51`,
  PC-USERS `192.168.20.51`, PC-IT `192.168.30.51`, PC-BRANCH `192.168.40.51`
  (see `verification/dhcp.md`).

---

## 8. NAT/PAT

### R1 (S3)

| Item | Value |
| --- | --- |
| Type | PAT, interface overload |
| Inside interfaces | Gi0/0.10, Gi0/0.20, Gi0/0.30, Gi0/0.50 (**S5**) |
| Outside interface | Gi0/2 (203.0.113.1/30) |
| Neither | Gi0/1 (R1–R2 transit) |
| NAT match ACL | `access-list 1`: permit 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24, **192.168.50.0/24** (S5) |

### R2 (S4)

| Item | Value |
| --- | --- |
| Type | PAT, interface overload |
| Inside interfaces | Gi0/1 (branch LAN), Gi0/0 (R1-facing transit) |
| Outside interface | Gi0/2 (203.0.113.5/30) |
| NAT match ACL | `access-list 1`: permit 192.168.40.0/24 (branch LAN) and HQ subnets 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24, **192.168.50.0/24** (S5) |

HQ-to-branch traffic leaves R1 through Gi0/1 (not outside) and reaches R2
through Gi0/0 and Gi0/1 (both inside), so it is not translated. On the backup
path, HQ traffic (including VLAN 50) enters R2 on Gi0/0 and leaves on Gi0/2, so
R2 translates it to `203.0.113.5`.

---

## 9. PC addressing

| Device | Addressing | Gateway | VLAN / LAN |
| --- | --- | --- | --- |
| PC-ADMIN | DHCP from R1 (S3); was static 192.168.10.10 | 192.168.10.1 | VLAN 10 |
| PC-USERS | DHCP from R1 (S3); was static 192.168.20.10 | 192.168.20.1 | VLAN 20 |
| PC-IT | DHCP from R1 (S3); was static 192.168.30.10 | 192.168.30.1 | VLAN 30 |
| PC-BRANCH | DHCP from R2 (S4); was static 192.168.40.10 | 192.168.40.1 | Branch LAN / VLAN 1 |

All four PCs receive DNS server `192.168.50.10` through DHCP (**S5**).

---

## 10. Address reservation

Reserved range in each client subnet: `.1`–`.50`.

Used for:
- Default gateways (`.1`)
- Temporary static test PCs (`.10`) in VLANs 10, 20, 30 and the branch, used
  before DHCP
- Infrastructure/server addresses: in VLAN 50, `.10` is the DNS server and
  `.11` is the intranet web server (**S5**)

DHCP dynamic ranges sit outside this reserved block.

---

## 11. Servers and DNS (**S5**)

### Server addressing

| Server | Address | Gateway | Location | Services |
| --- | --- | --- | --- | --- |
| DNS Server | 192.168.50.10/24 (static) | 192.168.50.1 | VLAN 50, SW3 Fa0/1 | DNS |
| Web Server | 192.168.50.11/24 (static) | 192.168.50.1 | VLAN 50, SW3 Fa0/2 | HTTP (intranet) |
| Public Web Server | 198.51.100.10/24 (static) | 198.51.100.1 | R3 Gi0/2 public server LAN | HTTP (simulated public site) |

### DNS records (on 192.168.50.10)

| Hostname | Address | Points to |
| --- | --- | --- |
| intranet.local | 192.168.50.11 | Web Server (internal) |
| publicwebsite.com | 198.51.100.10 | Public Web Server (simulated) |

- Clients learn `192.168.50.10` as their DNS server through DHCP (section 7).
- `publicwebsite.com` resolves only in this lab; it is not a real Internet name.
- Tests use the hostnames, not only the IP addresses (see `verification/dns.md`).

---

## 12. STP (**S5**)

PVST+ (the 2960 default) keeps a separate spanning tree per VLAN. No VTP is used.

| Switch | Role | Configuration |
| --- | --- | --- |
| SW1 | Root primary | `spanning-tree vlan 10,20,30,50 priority 24576` |
| SW2 | Root secondary | `spanning-tree vlan 10,20,30,50 priority 28672` |
| SW3 | Non-root | Default priority (32768) |

Verified in `verification/stp.md` for each of VLANs 10, 20, 30 and 50:
- SW1 is the root bridge (priority 24576 plus the VLAN ID).
- SW2 and SW3 both use Fa0/23 as the root port toward SW1.
- SW3 Fa0/24 (to SW2) is the alternate (blocking) port, because SW2 has the
  lower bridge ID.

VLAN 1 is not part of the design and stays at its default priority. SW3, which
has the lowest MAC address, is its root.

---

## 13. Open items

- None for Session 5.