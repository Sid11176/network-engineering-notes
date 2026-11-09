# IP and VLAN Plan

Last updated: end of Session 2 (`v0-foundation.pkt` complete)

Scope: Sessions 1–2 only.
Status: All entries below are Active in `v0-foundation.pkt`.
Later-session items (DHCP, NAT, ISP/R3, VLAN 50, VLAN 99, STP triangle,
servers, hardening) are intentionally omitted.

---

## 1. VLAN plan

| Site | VLAN | Name | Subnet | Default gateway |
| --- | --- | --- | --- | --- |
| HQ | 10 | ADMIN | 192.168.10.0/24 | 192.168.10.1 |
| HQ | 20 | USERS | 192.168.20.0/24 | 192.168.20.1 |
| HQ | 30 | IT | 192.168.30.0/24 | 192.168.30.1 |
| Branch | 1 | LAN BRANCH (default VLAN) | 192.168.40.0/24 | 192.168.40.1 |

Notes:
- No VTP is used. VLANs are created manually.
- At Session 2 the branch is a single flat LAN, so SW4 uses VLAN 1.
- R1 Gi0/0 carries no IP address; it is only the 802.1Q trunk parent.

---

## 2. Device inventory

| Device | Model | Role |
| --- | --- | --- |
| R1 | 2911 | HQ router, router-on-a-stick |
| R2 | 2911 | Branch router |
| SW1 | 2960 | HQ access switch |
| SW4 | 2960 | Branch access switch |
| PC-ADMIN | PC | VLAN 10 client |
| PC-USERS | PC | VLAN 20 client |
| PC-IT | PC | VLAN 30 client |
| PC-BRANCH | PC | Branch client |

---

## 3. Interface and addressing map

### R1 — HQ router

| Interface | Encapsulation | IP address | Purpose |
| --- | --- | --- | --- |
| Gi0/0 | — | no IP | Physical trunk to SW1 Gi0/1 |
| Gi0/0.10 | dot1Q 10 | 192.168.10.1/24 | VLAN 10 gateway |
| Gi0/0.20 | dot1Q 20 | 192.168.20.1/24 | VLAN 20 gateway |
| Gi0/0.30 | dot1Q 30 | 192.168.30.1/24 | VLAN 30 gateway |
| Gi0/1 | — | 10.0.0.1/30 | Transit link to R2 Gi0/0 |
| Gi0/2 | — | unconfigured | Unused in Sessions 1–2 |

### R2 — Branch router

| Interface | IP address | Purpose |
| --- | --- | --- |
| Gi0/0 | 10.0.0.2/30 | Transit link to R1 Gi0/1 |
| Gi0/1 | 192.168.40.1/24 | Branch LAN gateway to SW4 |
| Gi0/2 | unconfigured | Unused in Sessions 1–2 |

### SW1 — HQ switch

| Interface | Mode | VLAN / allowed VLANs | Purpose |
| --- | --- | --- | --- |
| Gi0/1 | trunk | allowed 10, 20, 30 | Uplink to R1 Gi0/0 |
| Fa0/1 | access | VLAN 10 | PC-ADMIN |
| Fa0/2 | access | VLAN 20 | PC-USERS |
| Fa0/3 | access | VLAN 30 | PC-IT |
| Fa0/4–Fa0/24 | access | VLAN 1 | Unused in Sessions 1–2 |

### SW4 — Branch switch

| Interface | Mode | VLAN | Purpose |
| --- | --- | --- | --- |
| Gi0/1 | access | VLAN 1 | Uplink to R2 Gi0/1 |
| Fa0/1 | access | VLAN 1 | PC-BRANCH |
| Fa0/2–Fa0/24 | access | VLAN 1 | Unused in Sessions 1–2 |

---

## 4. Physical link map

| From | Port | To | Port | Cable | Link type |
| --- | --- | --- | --- | --- | --- |
| R1 | Gi0/0 | SW1 | Gi0/1 | straight-through | 802.1Q trunk, allowed VLANs 10, 20, 30 |
| SW1 | Fa0/1 | PC-ADMIN | NIC | straight-through | Access, VLAN 10 |
| SW1 | Fa0/2 | PC-USERS | NIC | straight-through | Access, VLAN 20 |
| SW1 | Fa0/3 | PC-IT | NIC | straight-through | Access, VLAN 30 |
| R1 | Gi0/1 | R2 | Gi0/0 | crossover | Routed /30 |
| R2 | Gi0/1 | SW4 | Gi0/1 | straight-through | Access, VLAN 1 |
| SW4 | Fa0/1 | PC-BRANCH | NIC | straight-through | Access, VLAN 1 |

---

## 5. Routed links

| Link | Subnet | Endpoint A | Endpoint B |
| --- | --- | --- | --- |
| R1–R2 transit | 10.0.0.0/30 | R1 Gi0/1: 10.0.0.1 | R2 Gi0/0: 10.0.0.2 |
| Branch LAN | 192.168.40.0/24 | R2 Gi0/1: 192.168.40.1 | SW4 / PC-BRANCH |

---

## 6. Static routing

| Router | Destination | Next hop | Purpose |
| --- | --- | --- | --- |
| R1 | 192.168.40.0/24 | 10.0.0.2 | Route to branch LAN |
| R2 | 192.168.10.0/24 | 10.0.0.1 | Route to HQ VLAN 10 |
| R2 | 192.168.20.0/24 | 10.0.0.1 | Route to HQ VLAN 20 |
| R2 | 192.168.30.0/24 | 10.0.0.1 | Route to HQ VLAN 30 |

No default route, floating route, or ISP route exists in Sessions 1–2.

---

## 7. Temporary PC static addressing

| Device | IP address | Mask | Default gateway | VLAN / LAN |
| --- | --- | --- | --- | --- |
| PC-ADMIN | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | VLAN 10 |
| PC-USERS | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | VLAN 20 |
| PC-IT | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 | VLAN 30 |
| PC-BRANCH | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 | Branch LAN / VLAN 1 |

These are temporary static addresses. DHCP is not configured in Sessions 1–2.

---

## 8. Address reservation

Reserved range in each subnet: `.1`–`.50`.

Used for:
- Default gateways (`.1`)
- Temporary test PCs (`.10`)
- Future infrastructure/server addresses (not configured yet)

DHCP pools are not part of Sessions 1–2.

---

## 9. Open items

- Confirm actual Packet Tracer interface names on R1, R2, SW1, and SW4.
  If they differ, update this file and the configs to match the real ports.
- Keep `docs/ip-plan.md`, the `.pkt` file, and the saved configs consistent
  before committing `v0-foundation.pkt`.