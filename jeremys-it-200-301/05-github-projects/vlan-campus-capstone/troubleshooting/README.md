# Troubleshooting Index

Six faults injected into a clean copy of `v2-standout.pkt`, one change per fault. Each record uses the same fields: symptom, scope, layer, commands/tests, root cause, fix, verification.

| # | Fault injected | Layer | What the user sees | Key evidence | Root cause | Record |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | VLAN 50 removed from the SW1–R1 trunk | L2 | VLAN 50 servers and hostnames unreachable from other VLANs and the branch; Internet works | `show interfaces trunk` on SW1: `Gi0/1` allows only 10, 20, 30; no VLAN 50 ARP entries on R1 | VLAN 50 missing from the trunk's allowed list | [01-trunk-failure.md](01-trunk-failure.md) |
| 2 | R1 default route removed (two stages) | L3 | Stage 1: nothing visible, path changes. Stage 2: HQ cannot reach the Internet | `show ip route` on R1: floating default installed (Stage 1), then no gateway of last resort (Stage 2) | Primary default removed, then the floating backup as well | [02-default-route-failure.md](02-default-route-failure.md) |
| 3 | R1 `Gi0/0` shut down | L1/L2 (router side) | No HQ client reaches its gateway; all four HQ VLANs affected | `show ip interface brief` on R1: `Gi0/0` and the `Gi0/0.x` subinterfaces down | `shutdown` on the ROAS trunk parent | [03-roas-interface-failure.md](03-roas-interface-failure.md) |
| 4 | Floating default removed, then R1 `Gi0/2` shut | L3 | Works until the primary link fails; then HQ cannot reach the Internet | `show ip route` shows no gateway of last resort; `show running-config \| include ip route` shows no AD 10 line | Backup route missing, so no failover | [04-floating-route-failure.md](04-floating-route-failure.md) |
| 5 | SW3 priority 4096 for VLAN 20 | L2 (STP) | No outage; VLAN 20 takes a longer path | `show spanning-tree vlan 20`: SW3 is root (4116), SW2 `Fa0/23` is Altn BLK | Wrong root bridge for VLAN 20 | [05-stp-root-placement.md](05-stp-root-placement.md) |
| 6 | `intranet.local` DNS record removed | Application (DNS) | `intranet.local` does not resolve; the server answers by IP | `nslookup` returns "Non-existent domain"; `publicwebsite.com` still resolves | Missing A record on the DNS server | [06-dns-record-failure.md](06-dns-record-failure.md) |

## Quick triage

Match the symptom first, then open the record.

| If you see | Check first | Likely case |
| --- | --- | --- |
| One VLAN's servers unreachable, gateway interface up/up | `show interfaces trunk` | 1 |
| All HQ VLANs lose their gateway at once | `show ip interface brief` on R1 | 3 |
| Internet fails but internal traffic works | `show ip route` on R1 | 2, 4 |
| Internet works only until a link fails | `show running-config \| include ip route` on R1 | 4 |
| Everything works but the path looks wrong | `show spanning-tree vlan <id>` on each switch | 5 |
| IP works but a name does not resolve | `nslookup`, then compare with a second name | 6 |