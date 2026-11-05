# Verification — Capstone-Lite

Every stage was verified with `show` commands and end-to-end pings before moving
on. All checks below passed. Screenshots are in `screenshots/`.

## Stage 1 — Core

### VLANs and access ports

| Device | Command            | Expected                                          | Result |
|--------|--------------------|---------------------------------------------------|--------|
| SW1    | `show vlan brief`  | VLANs 10,20,30 active; F0/1=10, F0/2=20, F0/3=30   | ✅ pass |
| SW2    | `show vlan brief`  | VLANs 10,20,30 active; F0/1=10, F0/2=20, F0/3=30   | ✅ pass |

### Trunks

| Device | Command                 | Expected                                            | Result |
|--------|-------------------------|-----------------------------------------------------|--------|
| SW1    | `show interfaces trunk` | G0/1, G0/2, F0/23 trunking; VLANs 10,20,30 allowed  | ✅ pass |
| SW2    | `show interfaces trunk` | G0/1, F0/23 trunking; VLANs 10,20,30 allowed        | ✅ pass |

### STP

| Device | Command                      | Expected                       | Result |
|--------|------------------------------|--------------------------------|--------|
| SW1    | `show spanning-tree vlan 10` | "This bridge is the root"      | ✅ pass |
| SW2    | `show spanning-tree vlan 10` | G0/1 Root/FWD, F0/23 Altn/BLK  | ✅ pass |

### Router-on-a-stick

| Device | Command                   | Expected                                                  | Result |
|--------|---------------------------|-----------------------------------------------------------|--------|
| R1     | `show ip interface brief` | G0/0 up/up unassigned; .10/.20/.30 up/up with gateway IPs | ✅ pass |
| R1     | `show ip route`           | 3 connected: 192.168.10.0/24, .20.0/24, .30.0/24          | ✅ pass |

### Stage 1 connectivity tests

Each ping was run twice; first attempt may drop packets to ARP.

| # | From | To            | Proves                                 | Result |
|---|------|---------------|----------------------------------------|--------|
| 1 | PC1  | 192.168.10.1  | Default gateway reachable              | ✅ pass |
| 2 | PC1  | 192.168.10.11 | Same VLAN across switches (Trunk A)    | ✅ pass |
| 3 | PC1  | 192.168.20.10 | Inter-VLAN routing (ROAS)              | ✅ pass |
| 4 | PC1  | 192.168.30.10 | Inter-VLAN routing                     | ✅ pass |
| 5 | PC4  | 192.168.20.10 | Cross-switch + inter-VLAN              | ✅ pass |
| 6 | PC6  | 192.168.10.10 | Full end-to-end                        | ✅ pass |

### STP failover test

| Step | Action                                | Expected                        | Result |
|------|---------------------------------------|---------------------------------|--------|
| 1    | `show spanning-tree vlan 10` on SW2   | G0/1 FWD, F0/23 BLK             | ✅ pass |
| 2    | `ping -t 192.168.10.11` from PC1      | Replies flowing                 | ✅ pass |
| 3    | Delete SW1 G0/2 ↔ SW2 G0/1 cable      | Short outage, then pings resume | ✅ pass |
| 4    | `show spanning-tree vlan 10` on SW2   | F0/23 now Root/FWD              | ✅ pass |
| 5    | Reconnect Gig cable                   | F0/23 returns to BLK            | ✅ pass |

**Observed:** a brief burst of dropped pings during reconvergence, then full
recovery without any config change — confirming the Fast link works as a
standby path.

## Stage 2 — Static Routing

### Routing tables

| Device | Command          | Expected                                                    | Result |
|--------|------------------|-------------------------------------------------------------|--------|
| R1     | `show ip route`  | `S 192.168.40.0/24 [1/0] via 10.0.0.2`                      | ✅ pass |
| R2     | `show ip route`  | 3× `S` via 10.0.0.1; 2× `C` (10.0.0.0/30, 192.168.40.0/24)  | ✅ pass |

### Remote connectivity

| From | To            | Expected | Result |
|------|---------------|----------|--------|
| PC1  | 192.168.40.10 | reply    | ✅ pass |
| PC7  | 192.168.10.10 | reply    | ✅ pass |

### Path verification

| From | Command                 | Expected hops                           | Result |
|------|-------------------------|-----------------------------------------|--------|
| PC1  | `tracert 192.168.40.10` | 192.168.10.1 → 10.0.0.2 → 192.168.40.10 | ✅ pass |

## Summary

- **VLANs, access ports, trunks** — verified with `show vlan brief` / `show interfaces trunk`
- **STP root election and blocking** — verified with `show spanning-tree vlan 10`
- **Inter-VLAN routing** — verified by cross-VLAN pings
- **Redundancy** — verified by physical cable pull and reconvergence
- **Remote reachability** — verified by bidirectional pings + `tracert` path

All Stage 1 and Stage 2 checks passed. Device configurations saved
(`R1`, `R2`, `SW1`, `SW2`) and the Packet Tracer topology (`.pkt`) archived in
`artifacts/`.