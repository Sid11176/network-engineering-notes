# Troubleshooting Case 3: R1 ROAS interface shut down

Network: `v2-standout.pkt` (restored clean copy, saved as `v2-case3.pkt`). One change per fault. Session 6.

## Status

| Step | Status | Evidence |
| --- | --- | --- |
| Baseline | Captured | Section 1 |
| Fault injected | Captured | Section 2 |
| Symptoms observed | Captured | Section 3 |
| Diagnosis | Captured | Section 4 |
| Fix and recovery | Captured | Section 5 |

## Summary

| Item | Record |
| --- | --- |
| **Symptom** | No HQ client can reach its default gateway, so nothing leaves its VLAN. HQ clients cannot reach the Internet, the servers in VLAN 50, or the branch. |
| **Scope** | All four HQ VLANs (10, 20, 30, 50) at once. Not affected: traffic inside a VLAN (DNS to web server), R1's links to R2 and R3, and the branch's own Internet access. |
| **Layer** | Layer 1/2 on the router side. The physical trunk parent `Gi0/0` is administratively down, which takes every `Gi0/0.x` subinterface down with it. |
| **Root cause** | `shutdown` on R1 `Gi0/0`, the physical interface that carries the 802.1Q trunk to SW1 `Gi0/1`. |
| **Fix** | `no shutdown` on R1 `Gi0/0`. |
| **Verification** | The subinterfaces return to up/up, every HQ client reaches its gateway, and inter-VLAN, server, Internet and branch tests pass again (Section 5). |

## 1. Baseline

PC-ADMIN: `ping 192.168.10.1`, `ping 192.168.50.10`, `ping 8.8.8.8`.

```
C:\>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:

Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=1ms TTL=255

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

```
C:\>ping 192.168.50.10

Pinging 192.168.50.10 with 32 bytes of data:

Request timed out.
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127
Reply from 192.168.50.10: bytes=32 time=1ms TTL=127

Ping statistics for 192.168.50.10:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Request timed out.
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

R1: `show ip interface brief`

```
R1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES NVRAM  up                    up 
GigabitEthernet0/0.10  192.168.10.1    YES NVRAM  up                    up 
GigabitEthernet0/0.20  192.168.20.1    YES NVRAM  up                    up 
GigabitEthernet0/0.30  192.168.30.1    YES NVRAM  up                    up 
GigabitEthernet0/0.50  192.168.50.1    YES NVRAM  up                    up 
GigabitEthernet0/1     10.0.0.1        YES NVRAM  up                    up 
GigabitEthernet0/2     203.0.113.1     YES NVRAM  up                    up 
Vlan1                  unassigned      YES NVRAM  administratively down down
```

Expected: `Gi0/0` and `Gi0/0.10`, `.20`, `.30`, `.50` all up/up.

SW1: `show interfaces trunk`

```
SW1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/23      on           802.1q         trunking      1
Fa0/24      on           802.1q         trunking      1
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50
Gig0/1      10,20,30,50

Port        Vlans allowed and active in management domain
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50
Gig0/1      10,20,30,50

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50
Gig0/1      10,20,30,50
```

Expected: `Gig0/1` trunking, allowed VLANs 10, 20, 30, 50.

## 2. Fault injected

R1:

```
R1(config)#interface GigabitEthernet0/0
R1(config-if)#shutdown
```

```
R1#show running-config | section GigabitEthernet0/0
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
 shutdown
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip nat inside
interface GigabitEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.168.50.1 255.255.255.0
 ip nat inside
```

## 3. Symptoms observed

| Test | Expected result | Observed |
| --- | --- | --- |
| PC-ADMIN `ping 192.168.10.1` | Fails | FAIL |
| PC-USERS `ping 192.168.20.1` | Fails | FAIL |
| PC-IT `ping 192.168.30.1` | Fails | FAIL |
| DNS server `ping 192.168.50.1` | Fails | FAIL |
| DNS server `ping 192.168.50.11` (same VLAN, SW3 only) | Still succeeds | SUCCEED |
| PC-ADMIN `ping 192.168.50.10` (inter-VLAN) | Fails | FAIL |
| PC-ADMIN `ping 8.8.8.8` | Fails | FAIL |
| PC-BRANCH `ping 8.8.8.8` | Still succeeds (R2's own default route) | SUCCEED |
| PC-BRANCH `ping 192.168.10.51` (branch to HQ) | Fails | FAIL |
| R1 `ping 8.8.8.8` and `ping 10.0.0.2` | Both succeed | SUCCEED |

PC-ADMIN:

```
C:\>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

PC-USERS:

```
C:\>ping 192.168.20.1

Pinging 192.168.20.1 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.20.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

PC-IT:

```
C:\>ping 192.168.30.1

Pinging 192.168.30.1 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.30.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

DNS server:

```
C:\>ping 192.168.50.1

Pinging 192.168.50.1 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.50.1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

C:\>ping 192.168.50.11

Pinging 192.168.50.11 with 32 bytes of data:

Reply from 192.168.50.11: bytes=32 time<1ms TTL=128
Reply from 192.168.50.11: bytes=32 time<1ms TTL=128
Reply from 192.168.50.11: bytes=32 time<1ms TTL=128
Reply from 192.168.50.11: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.50.11:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

PC-ADMIN:

```
C:\>ping 192.168.50.10

Pinging 192.168.50.10 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.50.10:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),

C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

PC-BRANCH:

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Request timed out.
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.10.51

Pinging 192.168.10.51 with 32 bytes of data:

Request timed out.
Request timed out.
Request timed out.
Request timed out.

Ping statistics for 192.168.10.51:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

R1:

```
R1#ping 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms

R1#ping 10.0.0.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.0.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms
```

## 4. Diagnosis

1. **Scope.** The gateway of every HQ VLAN fails at once, while traffic inside a VLAN and the WAN links still work. That points to the one component all four VLANs share: R1's trunk-facing interface. In Case 1 only VLAN 50 failed, which pointed to a per-VLAN cause.
2. **Interface check.** `Gi0/0` is administratively down, and the subinterfaces are down with it.

```
R1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES NVRAM  administratively down down 
GigabitEthernet0/0.10  192.168.10.1    YES NVRAM  administratively down down 
GigabitEthernet0/0.20  192.168.20.1    YES NVRAM  administratively down down 
GigabitEthernet0/0.30  192.168.30.1    YES NVRAM  administratively down down 
GigabitEthernet0/0.50  192.168.50.1    YES NVRAM  administratively down down 
GigabitEthernet0/1     10.0.0.1        YES NVRAM  up                    up 
GigabitEthernet0/2     203.0.113.1     YES NVRAM  up                    up 
Vlan1                  unassigned      YES NVRAM  administratively down down
```

Expected: `Gi0/0` and the four subinterfaces all show administratively down/down, inherited from the parent. `Gi0/1` and `Gi0/2` stay up/up.

3. **Route table.** The connected routes for the HQ VLANs are gone because their interfaces are down.

```
R1#show ip route 
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 203.0.113.2 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/1
L       10.0.0.1/32 is directly connected, GigabitEthernet0/1
S    192.168.40.0/24 [1/0] via 10.0.0.2
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/2
L       203.0.113.1/32 is directly connected, GigabitEthernet0/2
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

Expected: no `C` or `L` entries for `192.168.10.0/24`, `192.168.20.0/24`, `192.168.30.0/24` or `192.168.50.0/24`. The routes for `10.0.0.0/30`, `203.0.113.0/30`, the branch static route and the default route are still present.

4. **Switch side.** SW1 `Gi0/1` loses its link, so it is no longer trunking.

```
SW1#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/23      on           802.1q         trunking      1
Fa0/24      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50

Port        Vlans allowed and active in management domain
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50
```

Expected: `Gig0/1` is missing from the list. Only `Fa0/23` and `Fa0/24` remain.

## 5. Fix and verification

R1:

```
R1(config)#interface GigabitEthernet0/0
R1(config-if)#no shutdown
```

```
R1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES NVRAM  up                    up 
GigabitEthernet0/0.10  192.168.10.1    YES NVRAM  up                    up 
GigabitEthernet0/0.20  192.168.20.1    YES NVRAM  up                    up 
GigabitEthernet0/0.30  192.168.30.1    YES NVRAM  up                    up 
GigabitEthernet0/0.50  192.168.50.1    YES NVRAM  up                    up 
GigabitEthernet0/1     10.0.0.1        YES NVRAM  up                    up 
GigabitEthernet0/2     203.0.113.1     YES NVRAM  up                    up 
Vlan1                  unassigned      YES NVRAM  administratively down down
```

Expected: `Gi0/0` and all four subinterfaces up/up.

```
SW1#show interfaces trunk 
Port        Mode         Encapsulation  Status        Native vlan
Fa0/23      on           802.1q         trunking      1
Fa0/24      on           802.1q         trunking      1
Gig0/1      on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50
Gig0/1      10,20,30,50

Port        Vlans allowed and active in management domain
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50
Gig0/1      10,20,30,50

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/23      10,20,30,50
Fa0/24      10,20,30,50
Gig0/1      10,20,30,50
```

The first ping may time out while ARP resolves.

PC-ADMIN:

```
C:\>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:

Request timed out.
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

PC-USERS and PC-IT:

```
C:\>ping 192.168.20.1

Pinging 192.168.20.1 with 32 bytes of data:

Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.20.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms


C:\>ping 192.168.30.1

Pinging 192.168.30.1 with 32 bytes of data:

Reply from 192.168.30.1: bytes=32 time<1ms TTL=255
Reply from 192.168.30.1: bytes=32 time<1ms TTL=255
Reply from 192.168.30.1: bytes=32 time<1ms TTL=255
Reply from 192.168.30.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.30.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

PC-ADMIN:

```
C:\>ping 192.168.50.10

Pinging 192.168.50.10 with 32 bytes of data:

Request timed out.
Reply from 192.168.50.10: bytes=32 time=11ms TTL=127
Reply from 192.168.50.10: bytes=32 time=2ms TTL=127
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.50.10:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 11ms, Average = 4ms

C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time=10ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 2ms
```

PC-ADMIN:

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   0 ms      1 ms      0 ms      8.8.8.8

Trace complete.
```

Expected: `192.168.10.1` then `8.8.8.8`.

PC-BRANCH:

```
C:\>ping 192.168.10.51

Pinging 192.168.10.51 with 32 bytes of data:

Reply from 192.168.10.51: bytes=32 time<1ms TTL=126
Reply from 192.168.10.51: bytes=32 time=10ms TTL=126
Reply from 192.168.10.51: bytes=32 time<1ms TTL=126
Reply from 192.168.10.51: bytes=32 time=1ms TTL=126

Ping statistics for 192.168.10.51:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 2ms
```

## Conclusion

- Configured: the fault (`shutdown` on R1 `Gi0/0`) and the fix (`no shutdown`).
- Verified: the captures above. Before the fix every HQ gateway was unreachable. After the fix the gateways, inter-VLAN routing, the servers, the Internet and the branch all work again.
- Explained:
  - **Why every VLAN failed at once:** with router-on-a-stick, all four HQ gateways are subinterfaces of the single physical port `Gi0/0`. Shutting the parent takes every subinterface down, so every connected route disappears.
  - **Why this differs from Case 1:** there, the subinterface stayed up/up and only VLAN 50 frames were dropped at the switch. Here the interface state itself shows the fault, and `show ip interface brief` finds it immediately.
  - **Why traffic inside a VLAN still worked:** DNS and web are both on SW3 in VLAN 50, so their traffic never needs the router.
  - **Why the branch and the WAN links were unaffected:** `Gi0/1` and `Gi0/2` are separate interfaces and stayed up. R1 could still reach R2 and R3, and PC-BRANCH reaches `8.8.8.8` through R2's own default route.
  - **Why the branch could not reach HQ:** R2 still sends HQ traffic to R1, but R1 has lost its connected routes for the HQ VLANs, so it cannot deliver it. The only route R1 has for those addresses is the default route toward R3, which has no route to private subnets.
  - **Lesson:** when all VLANs behind one trunk fail together, check the trunk-facing interfaces on both ends first: the router's `show ip interface brief` and the switch's `show interfaces trunk`.