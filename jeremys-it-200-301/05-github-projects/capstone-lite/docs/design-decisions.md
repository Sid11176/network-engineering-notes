# Design Decisions — Capstone-Lite

## Why three VLANs?

Segmentation. Three groups (ADMIN, USERS, IT) get separate broadcast domains and
separate subnets, which shrinks broadcast scope and gives a natural point to
apply policy later (ACLs, port security). Flat networks don't scale and don't
let you apply different treatment per group.

## Why router-on-a-stick instead of a Layer 3 switch?

The 2960 is a Layer 2 switch — no `ip routing`, no SVIs. ROAS is the correct
way to get inter-VLAN routing with the hardware available: one physical link
carrying 802.1Q-tagged frames, one subinterface per VLAN, each subinterface IP
becoming that VLAN's default gateway.

**Trade-off:** all inter-VLAN traffic funnels through one router interface, so
that link is a bandwidth and failure bottleneck. A Layer 3 switch with SVIs
would route at wire speed and remove the single uplink dependency.

## Why restrict trunk allowed VLANs to 10,20,30?

Two reasons: security and broadcast hygiene. Only VLANs that actually need to
cross the link should be tagged across it. Left at default, a trunk allows all
VLANs, so a future VLAN would silently propagate everywhere. Explicit lists also
make `show interfaces trunk` a meaningful verification tool.

## Why two inter-switch links if STP blocks one?

Redundancy. Without the second link, losing Trunk A partitions the network —
SW2's hosts lose their gateway. With both links, STP blocks the slower one and
keeps it as a hot standby. The cost is that you carry a deliberately unused
link; the benefit is that a cable or port failure is a convergence event, not
an outage.

## Why is SW1 the root bridge?

`spanning-tree vlan 10,20,30 root primary` on SW1 and `root secondary` on SW2
makes the root deterministic instead of leaving it to MAC-address tie-breaking.
SW1 is the natural choice because it is the switch closest to the router — the
inter-VLAN path. Keeping the root there means traffic between VLANs and toward
R1 takes the shortest Layer 2 path, rather than hairpinning through SW2.

## Why does G0/1 win and F0/23 block?

STP path cost is derived from link speed:

| Link speed | STP cost |
|------------|----------|
| Gigabit    | 4        |
| FastEthernet | 19     |

From SW2, the path to the root via G0/1 costs 4; via F0/23 it costs 19. Lower
cost wins, so G0/1 becomes the root port (forwarding) and F0/23 becomes the
alternate port (blocking). The slower link is the backup — which is exactly
what you want.

## Why PVST rather than rapid-PVST?

The lab targets classic PVST behavior so that the blocking/forwarding states
and the ~30–50 s convergence during the failover exercise are observable. In
production, rapid-PVSTP would be the better choice for faster convergence.

## Why static routes instead of a routing protocol?

Scope. There are exactly two routers and one stub network beyond R2. A routing
protocol would add configuration and convergence behavior for no benefit at
this size. Static routes are explicit and easy to verify with `show ip route`.

## Why routes in both directions?

Routing is hop-by-hop and stateless in the forwarding path. R1 knowing how to
reach 192.168.40.0/24 gets the echo request to PC7, but R2 must independently
know how to reach 192.168.10.0/24 to send the reply back. A one-way route
produces the classic symptom: request leaves, reply never returns.

## Why a /30 on the R1–R2 link?

A point-to-point link needs exactly two addresses. /30 provides two usable
hosts and nothing wasted. A /24 here would burn 254 addresses for no reason.

## Known limitations

- **R1 is a single point of failure** for all inter-VLAN and remote traffic. No
  HSRP/VRRP redundancy.
- **No dynamic routing** — adding a third site would mean manual route entry on
  every router.
- **Native VLAN left at default (VLAN 1)** — acceptable in a lab, but a
  production design should move the native VLAN to an unused VLAN and avoid
  using VLAN 1 for anything.
- **No access control** — all VLANs can reach all VLANs. No ACLs applied.
- **No port security, DHCP snooping, or Dynamic ARP Inspection.**
- **STP convergence is slow** (~30–50 s) because PVST, not rapid-PVSTP.
- **No management VLAN / out-of-band access.**

## Possible next steps

1. HSRP between two routers for gateway redundancy.
2. Replace static routes with OSPF.
3. ACLs to restrict IT VLAN ↔ USERS VLAN traffic.
4. Port security on access ports (sticky MAC, violation shutdown).
5. Move native VLAN off VLAN 1 and prune unused VLANs from trunks.
6. Migrate to rapid-PVSTP and compare convergence times.