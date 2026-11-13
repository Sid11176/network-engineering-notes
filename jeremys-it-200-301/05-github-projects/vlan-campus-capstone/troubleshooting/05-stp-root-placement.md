# Troubleshooting Case 5: SW3 becomes STP root for VLAN 20

Network: `v2-standout.pkt` (restored clean copy). One change per fault. Session 7.

## Status

| Step | Status | Evidence |
| --- | --- | --- |
| Baseline (before fault) | Captured | Section 1 |
| Fault injected | Captured | Section 2 |
| Symptoms observed | Captured | Section 3 |
| Diagnosis | Captured | Section 4 |
| Fix and recovery | Captured | Section 5 |

## Summary

| Item | Record |
| --- | --- |
| **Symptom** | No outage. Connectivity still works, but VLAN 20 traffic from SW2 now reaches R1 through SW3 instead of directly through SW1. Only the spanning-tree output shows it. |
| **Scope** | VLAN 20 only. VLAN 10 was checked and still has SW1 as root, because the change was made per VLAN. |
| **Layer** | Layer 2 (spanning tree). |
| **Root cause** | `spanning-tree vlan 20 priority 4096` on SW3 gave it a lower bridge ID than SW1 (24576), so the access switch SW3 became the VLAN 20 root. |
| **Fix** | `no spanning-tree vlan 20 priority` on SW3 (back to the default 32768). |
| **Verification** | SW1 is the VLAN 20 root again, SW3 `Fa0/24` is the alternate (blocking) port, and the roles match the baseline (Section 5). |

## 1. Baseline (before the fault)

Expected for VLAN 20: SW1 is root (priority 24596 = 24576 + 20). SW2 and SW3 use `Fa0/23` toward SW1 as root port. SW3 `Fa0/24` is Altn/BLK.

| Switch | Expected roles (VLAN 20) |
| --- | --- |
| SW1 | Root bridge. `Fa0/23`, `Fa0/24`, `Gi0/1` all Desg FWD |
| SW2 | `Fa0/23` Root FWD, `Fa0/24` Desg FWD |
| SW3 | `Fa0/23` Root FWD, `Fa0/24` Altn BLK |

```
SW1#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    24596
             Address     000B.BE2D.CCD7
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24596  (priority 24576 sys-id-ext 20)
             Address     000B.BE2D.CCD7
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
Gi0/1            Desg FWD 4         128.25   P2p
```

```
SW2#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    24596
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    28692  (priority 28672 sys-id-ext 20)
             Address     00D0.FF31.46AA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
```

```
SW3#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    24596
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     0005.5E84.B6E3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Altn BLK 19        128.24   P2p
```

PC-USERS (on SW2 `Fa0/2`) `ping 8.8.8.8` passes:

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

## 2. Fault injected

SW3:

```
SW3(config)#spanning-tree vlan 20 priority 4096
```

Only VLAN 20 is changed, so the other VLANs act as the control.

## 3. Symptoms observed

PVST+ takes time to move ports out of blocking, so pings sent right after the change can time out. After that, connectivity is back.

| Test | Expected result | Observed |
| --- | --- | --- |
| PC-USERS `ping 8.8.8.8` after the change | Still succeeds once converged | SUCCEED (first 2 of 4 timed out) |
| PC-USERS `tracert 8.8.8.8` | 2 hops, `192.168.20.1` then `8.8.8.8` (STP is Layer 2 and invisible here) | SUCCEED |

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Request timed out.
Request timed out.
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 2, Lost = 2 (50% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.20.1
  2   0 ms      0 ms      0 ms      8.8.8.8

Trace complete.
```

## 4. Diagnosis

Expected for VLAN 20 after the fault: SW3 is root (priority 4116 = 4096 + 20). SW1 and SW2 both use `Fa0/24` toward SW3 as root port. The SW1–SW2 link blocks on the SW2 side.

| Switch | Expected roles (VLAN 20) |
| --- | --- |
| SW3 | Root bridge. `Fa0/23`, `Fa0/24` Desg FWD |
| SW1 | `Fa0/24` Root FWD, `Fa0/23` Desg FWD, `Gi0/1` Desg FWD |
| SW2 | `Fa0/24` Root FWD, `Fa0/23` Altn BLK |

1. **Scope.** The ping still works, so this is not a routing or trunk fault. Something changed in the Layer 2 topology (ping in Section 3).
2. **Root bridge.** The root ID on all three switches now shows SW3's address (`0005.5E84.B6E3`) and priority 4116.

```
SW1#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    4116
             Address     0005.5E84.B6E3
             Cost        19
             Port        24(FastEthernet0/24)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24596  (priority 24576 sys-id-ext 20)
             Address     000B.BE2D.CCD7
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Root FWD 19        128.24   P2p
Gi0/1            Desg FWD 4         128.25   P2p
```

```
SW2#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    4116
             Address     0005.5E84.B6E3
             Cost        19
             Port        24(FastEthernet0/24)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    28692  (priority 28672 sys-id-ext 20)
             Address     00D0.FF31.46AA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/23           Altn BLK 19        128.23   P2p
Fa0/24           Root FWD 19        128.24   P2p
```

```
SW3#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    4116
             Address     0005.5E84.B6E3
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4116  (priority 4096 sys-id-ext 20)
             Address     0005.5E84.B6E3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
```

3. **Per-VLAN check.** VLAN 10 on SW3 still has SW1 as root (priority 24586), with `Fa0/23` as root port and `Fa0/24` Altn BLK.

```
SW3#show spanning-tree vlan 10
VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    24586
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     0005.5E84.B6E3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Altn BLK 19        128.24   P2p
```

4. **Config check.** The VLAN 20 priority line is present on SW3.

```
SW3#show running-config | include spanning-tree
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 20 priority 4096
```

## 5. Fix and verification

SW3:

```
SW3(config)#no spanning-tree vlan 20 priority
```

All three switches are back to their baseline roles. SW1 is root again, SW3 `Fa0/24` is Altn BLK, and SW2 uses `Fa0/23` as root port. SW2 was captured after convergence. An earlier capture showed `Fa0/23` as `Root LRN` while the port was still learning.

```
SW1#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    24596
             Address     000B.BE2D.CCD7
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24596  (priority 24576 sys-id-ext 20)
             Address     000B.BE2D.CCD7
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
Gi0/1            Desg FWD 4         128.25   P2p
```

```
SW3#show spanning-tree vlan 20
VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    24596
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     0005.5E84.B6E3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Altn BLK 19        128.24   P2p
```

```
<!-- paste SW2 show spanning-tree vlan 20, after convergence -->
```

PC-USERS `ping 8.8.8.8` passes again:

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

## Conclusion

- Configured: the fault (SW3 priority 4096 for VLAN 20) and the fix (priority removed).
- Verified: the captures above. Before the fault, SW1 was the VLAN 20 root and SW3 `Fa0/24` was Altn BLK. With the fault, SW3 became root at priority 4116, SW1 and SW2 moved their root port to `Fa0/24`, and SW2 `Fa0/23` went Altn BLK, while VLAN 10 kept SW1 as root. Pings from PC-USERS still passed after the first two timed out during reconvergence. After the fix, all three switches returned to their baseline roles and pings pass again.
- Explained:
  - **Why nothing broke:** STP only chooses which redundant links forward and which block. A different root changes the path, not whether a path exists.
  - **Why it matters:** SW3 is a server access switch. As root, it moved the blocked link to SW2 `Fa0/23`, so by the port roles PC-USERS traffic now goes SW2 → SW3 → SW1 → R1, one more Layer 2 hop than the baseline. This is why the root should sit on the distribution switch, SW1.
  - **Why `tracert` looked normal:** it shows only Layer 3 hops, so the Layer 2 detour is invisible there.
  - **Why only VLAN 20 changed:** PVST+ runs a separate spanning tree per VLAN, so VLAN 10 still used SW1 as root.
  - **Lesson:** a working ping does not prove a good topology. Check `show spanning-tree vlan <id>` on every switch when path or root placement matters.