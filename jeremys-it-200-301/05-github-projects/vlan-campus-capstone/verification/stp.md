# STP Verification: v2-standout

Network: `v2-standout.pkt`. The three HQ switches form a triangle (SW1–SW2, SW1–SW3, SW2–SW3) running per-VLAN spanning tree (PVST+, the 2960 default). VLANs 10, 20, 30 and 50 are carried on every trunk.

## Design

| Switch | Role | Configuration | Expected priority (VLAN 10 / 20 / 30 / 50) |
| --- | --- | --- | --- |
| SW1 | Root primary | `spanning-tree vlan 10,20,30,50 priority 24576` (root primary) | 24586 / 24596 / 24606 / 24626 |
| SW2 | Root secondary | `spanning-tree vlan 10,20,30,50 priority 28672` (root secondary) | 28682 / 28692 / 28702 / 28722 |
| SW3 | Non-root | default | 32778 / 32788 / 32798 / 32818 |

The priority is the base value plus the VLAN ID (24576, 28672 and 32768 are the base values). A VLAN left out of the root commands would show 32768 plus its ID on every switch, so VLAN 50 is the one to check.

## Expected port roles (the same in every VLAN)

| Switch | Fa0/23 | Fa0/24 | Other ports |
| --- | --- | --- | --- |
| SW1 (root) | Desg FWD (to SW2) | Desg FWD (to SW3) | Gi0/1 to R1: Desg FWD |
| SW2 | Root FWD (to SW1) | Desg FWD (to SW3) | Access port of that VLAN: Desg FWD |
| SW3 | Root FWD (to SW1) | **Altn BLK** (to SW2) | VLAN 50 access ports Fa0/1, Fa0/2: Desg FWD |

SW3 blocks Fa0/24 because SW2 and SW3 have the same cost to the root, and SW2 wins the tie with the lower bridge ID.

## Evidence

### Root configuration

SW1: `show running-config | include spanning-tree`

```
SW1#show running-config | include spanning-tree
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 10,20,30,50 priority 24576
```

SW2: `show running-config | include spanning-tree`

```
SW2#show running-config | include spanning-tree
spanning-tree mode pvst
spanning-tree extend system-id
spanning-tree vlan 10,20,30,50 priority 28672
```

### Per-VLAN state

`show spanning-tree` prints every VLAN, so one command per switch covers VLANs 10, 20, 30 and 50. VLAN 1 also appears. It is not part of the design and stays at its default priority, so SW3 (lowest MAC address) is its root. This does not affect VLANs 10–50.

SW1: `show spanning-tree`

```
SW1#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E84.B6E3
             Cost        19
             Port        24(FastEthernet0/24)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     000B.BE2D.CCD7
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Root FWD 19        128.24   P2p
Gi0/1            Desg FWD 4         128.25   P2p

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    24586
             Address     000B.BE2D.CCD7
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24586  (priority 24576 sys-id-ext 10)
             Address     000B.BE2D.CCD7
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
Gi0/1            Desg FWD 4         128.25   P2p

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

VLAN0030
  Spanning tree enabled protocol ieee
  Root ID    Priority    24606
             Address     000B.BE2D.CCD7
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24606  (priority 24576 sys-id-ext 30)
             Address     000B.BE2D.CCD7
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
Gi0/1            Desg FWD 4         128.25   P2p

VLAN0050
  Spanning tree enabled protocol ieee
  Root ID    Priority    24626
             Address     000B.BE2D.CCD7
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    24626  (priority 24576 sys-id-ext 50)
             Address     000B.BE2D.CCD7
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
Gi0/1            Desg FWD 4         128.25   P2p
```

SW2: `show spanning-tree`

```
SW2#show spanning-tree
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E84.B6E3
             Cost        19
             Port        24(FastEthernet0/24)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     00D0.FF31.46AA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Altn BLK 19        128.23   P2p
Fa0/24           Root FWD 19        128.24   P2p

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    24586
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    28682  (priority 28672 sys-id-ext 10)
             Address     00D0.FF31.46AA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p

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

VLAN0030
  Spanning tree enabled protocol ieee
  Root ID    Priority    24606
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    28702  (priority 28672 sys-id-ext 30)
             Address     00D0.FF31.46AA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/3            Desg FWD 19        128.3    P2p
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p

VLAN0050
  Spanning tree enabled protocol ieee
  Root ID    Priority    24626
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    28722  (priority 28672 sys-id-ext 50)
             Address     00D0.FF31.46AA
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p
```

SW3: `show spanning-tree`

```
SW3#show spanning-tree 
VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     0005.5E84.B6E3
             This bridge is the root
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     0005.5E84.B6E3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Desg FWD 19        128.23   P2p
Fa0/24           Desg FWD 19        128.24   P2p

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

VLAN0030
  Spanning tree enabled protocol ieee
  Root ID    Priority    24606
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     0005.5E84.B6E3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Altn BLK 19        128.24   P2p

VLAN0050
  Spanning tree enabled protocol ieee
  Root ID    Priority    24626
             Address     000B.BE2D.CCD7
             Cost        19
             Port        23(FastEthernet0/23)
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32818  (priority 32768 sys-id-ext 50)
             Address     0005.5E84.B6E3
             Hello Time  2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  20

Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/2            Desg FWD 19        128.2    P2p
Fa0/1            Desg FWD 19        128.1    P2p
Fa0/23           Root FWD 19        128.23   P2p
Fa0/24           Altn BLK 19        128.24   P2p
```

## Conclusion

- Configured: SW1 as root primary and SW2 as root secondary for VLANs 10, 20, 30 and 50.
- Verified: in each of VLANs 10, 20, 30 and 50, SW1 is the root bridge, SW2's root port is Fa0/23, and SW3's Fa0/24 is the alternate (blocking) port. The root priority for each VLAN matches the table above, so VLAN 50 is covered by the root settings.
- Explained:
  - **Why the triangle needs STP:** three switches joined in a loop would forward frames around it forever (a broadcast storm). STP blocks one port so the topology is a loop-free tree, and the blocked link stays ready as a backup (tested in `failover.md`).
  - **Why VLANs can have different roles:** PVST+ runs a separate tree per VLAN, so roles follow each VLAN's own priorities, port costs and allowed trunks. They are identical here only because the same root settings and costs apply to every VLAN. A VLAN missing from the root commands, or missing from a trunk's allowed list, would get a different root or a different blocked port.