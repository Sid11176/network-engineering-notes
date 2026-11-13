# Troubleshooting Case 4: R1 floating default route missing

Network: `v2-standout.pkt` (restored clean copy). One change per fault. Session 7.

## Status

| Step | Status | Evidence |
| --- | --- | --- |
| Baseline (before fault) | Captured | Section 1 |
| Fault injected | Captured | Section 2 |
| Symptoms observed | Captured (failing Internet test from VLAN 10 only) | Section 3 |
| Diagnosis | Captured | Section 4 |
| Fix and recovery | Captured | Section 5 |

## Summary

| Item | Record |
| --- | --- |
| **Symptom** | While R1 Gi0/2 is up, nothing looks wrong. After Gi0/2 goes down, HQ clients cannot reach `8.8.8.8`: pings return "Destination host unreachable" from `192.168.10.1`. |
| **Scope** | HQ Internet traffic only (tested from VLAN 10). Inter-VLAN, HQ to branch, VLAN 50 servers, branch to HQ and the branch's own Internet access (R2's default via R3) still work. |
| **Layer** | Layer 3 (routing). |
| **Root cause** | The floating default route `ip route 0.0.0.0 0.0.0.0 10.0.0.2 10` was removed from R1. When the primary default (AD 1, via `203.0.113.2`) was withdrawn with Gi0/2, R1 had no default route left. |
| **Fix** | Re-add `ip route 0.0.0.0 0.0.0.0 10.0.0.2 10` on R1. |
| **Verification** | With Gi0/2 down, R1 installs the AD 10 default and `tracert 8.8.8.8` goes via `10.0.0.2`. After `no shutdown`, the AD 1 default returns (Section 5). |

## 1. Baseline (before the fault)

PC-ADMIN tests, all passing: `ping 8.8.8.8`, ping the branch PC (`192.168.40.x`), `ping 192.168.50.10`.

R1: `show running-config | include ip route`. Expected: primary default (AD 1) and floating default (AD 10) both configured.

```
R1#show running-config | include ip route
ip route 192.168.40.0 255.255.255.0 10.0.0.2 
ip route 0.0.0.0 0.0.0.0 203.0.113.2 
ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
```

R1: `show ip route`. Expected: only the AD 1 default is installed. The floating route is configured but not in the table.

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
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
     192.168.30.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.30.0/24 is directly connected, GigabitEthernet0/0.30
L       192.168.30.1/32 is directly connected, GigabitEthernet0/0.30
S    192.168.40.0/24 [1/0] via 10.0.0.2
     192.168.50.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.50.0/24 is directly connected, GigabitEthernet0/0.50
L       192.168.50.1/32 is directly connected, GigabitEthernet0/0.50
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/2
L       203.0.113.1/32 is directly connected, GigabitEthernet0/2
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

Only `[1/0] via 203.0.113.2` is installed. There is no `[10/0]` line, even though the floating route is configured.

## 2. Fault injected

R1:

```
R1(config)#no ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
R1(config)#interface gigabitEthernet 0/2
R1(config-if)#shutdown
```

Removing only the floating route changes nothing visible while the primary is up. The fault appears when Gi0/2 goes down.

## 3. Symptoms observed

| Test | Expected result | Observed |
| --- | --- | --- |
| PC-ADMIN `ping 8.8.8.8` | Fails | FAIL |
| PC-BRANCH `ping 8.8.8.8` | Still succeeds | SUCCEED |
| HQ inter-VLAN `ping 192.168.30.51` | Still succeeds | SUCCEED |
| HQ to branch `ping 192.168.40.51` | Still succeeds | SUCCEED |
| HQ to VLAN 50 `ping 192.168.50.10` | Still succeeds | SUCCEED |
| Branch to HQ `ping 192.168.20.51` | Still succeeds | SUCCEED |

PC-ADMIN to the Internet fails, and the unreachable reply comes from `192.168.10.1`, so R1 itself has no route:

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

PC-BRANCH reaches the Internet through R2's own default route:

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time=8ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 8ms, Average = 2ms
```

Internal destinations still work from HQ:

```
C:\>ping 192.168.30.51

Pinging 192.168.30.51 with 32 bytes of data:

Request timed out.
Reply from 192.168.30.51: bytes=32 time<1ms TTL=127
Reply from 192.168.30.51: bytes=32 time<1ms TTL=127
Reply from 192.168.30.51: bytes=32 time=10ms TTL=127

Ping statistics for 192.168.30.51:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 3ms
```

```
C:\>ping 192.168.40.51

Pinging 192.168.40.51 with 32 bytes of data:

Reply from 192.168.40.51: bytes=32 time=6ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time=33ms TTL=126
Reply from 192.168.40.51: bytes=32 time=1ms TTL=126

Ping statistics for 192.168.40.51:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 33ms, Average = 10ms
```

```
C:\>ping 192.168.50.10

Pinging 192.168.50.10 with 32 bytes of data:

Request timed out.
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.50.10:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

Branch to HQ still works:

```
C:\>ping 192.168.20.51

Pinging 192.168.20.51 with 32 bytes of data:

Request timed out.
Reply from 192.168.20.51: bytes=32 time=10ms TTL=126
Reply from 192.168.20.51: bytes=32 time=10ms TTL=126
Reply from 192.168.20.51: bytes=32 time=10ms TTL=126

Ping statistics for 192.168.20.51:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 10ms, Maximum = 10ms, Average = 3ms
```

The first request in several pings times out while ARP resolves. This is normal and not a fault. Only VLAN 10 was tested as the failing HQ client.

## 4. Diagnosis

1. **Scope.** Only HQ Internet traffic fails. Internal routing and the branch path work, so this is not a Layer 2 or NAT problem. PC-ADMIN can still reach its gateway:

```
C:\>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:

Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=4ms TTL=255

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 4ms, Average = 1ms
```

2. **Interface check.** Gi0/2 is administratively down. All `Gi0/0.x` subinterfaces and Gi0/1 are up/up.

```
R1#show ip interface brief 
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES NVRAM  up                    up 
GigabitEthernet0/0.10  192.168.10.1    YES manual up                    up 
GigabitEthernet0/0.20  192.168.20.1    YES manual up                    up 
GigabitEthernet0/0.30  192.168.30.1    YES manual up                    up 
GigabitEthernet0/0.50  192.168.50.1    YES manual up                    up 
GigabitEthernet0/1     10.0.0.1        YES NVRAM  up                    up 
GigabitEthernet0/2     203.0.113.1     YES NVRAM  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
```

3. **Routing table.** R1 has no gateway of last resort, and the `203.0.113.0/30` connected route is gone with Gi0/2.

```
R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/1
L       10.0.0.1/32 is directly connected, GigabitEthernet0/1
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
     192.168.30.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.30.0/24 is directly connected, GigabitEthernet0/0.30
L       192.168.30.1/32 is directly connected, GigabitEthernet0/0.30
S    192.168.40.0/24 [1/0] via 10.0.0.2
     192.168.50.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.50.0/24 is directly connected, GigabitEthernet0/0.50
L       192.168.50.1/32 is directly connected, GigabitEthernet0/0.50
```

4. **Running config.** Only the AD 1 default and the branch route remain. The AD 10 floating route is missing.

```
R1#show running-config | include ip route
ip route 192.168.40.0 255.255.255.0 10.0.0.2 
ip route 0.0.0.0 0.0.0.0 203.0.113.2 
```

5. **Branch check.** R2 is healthy and has its own default via R3.

```
R2#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is 203.0.113.6 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/0
L       10.0.0.2/32 is directly connected, GigabitEthernet0/0
S    192.168.10.0/24 [1/0] via 10.0.0.1
S    192.168.20.0/24 [1/0] via 10.0.0.1
S    192.168.30.0/24 [1/0] via 10.0.0.1
     192.168.40.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.40.0/24 is directly connected, GigabitEthernet0/1
L       192.168.40.1/32 is directly connected, GigabitEthernet0/1
S    192.168.50.0/24 [1/0] via 10.0.0.1
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.4/30 is directly connected, GigabitEthernet0/2
L       203.0.113.5/32 is directly connected, GigabitEthernet0/2
S*   0.0.0.0/0 [1/0] via 203.0.113.6
```

## 5. Fix and verification

R1 (Gi0/2 still down):

```
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
```

```
R1#show ip route
Gateway of last resort is 10.0.0.2 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/1
L       10.0.0.1/32 is directly connected, GigabitEthernet0/1
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
     192.168.30.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.30.0/24 is directly connected, GigabitEthernet0/0.30
L       192.168.30.1/32 is directly connected, GigabitEthernet0/0.30
S    192.168.40.0/24 [1/0] via 10.0.0.2
     192.168.50.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.50.0/24 is directly connected, GigabitEthernet0/0.50
L       192.168.50.1/32 is directly connected, GigabitEthernet0/0.50
S*   0.0.0.0/0 [10/0] via 10.0.0.2
```

PC-ADMIN:

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   0 ms      0 ms      0 ms      10.0.0.2
  3   0 ms      10 ms     10 ms     8.8.8.8

Trace complete.
```

R2 translates the HQ traffic on the backup path:

```
R2#show ip nat translations 
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.5:15    192.168.10.51:15   8.8.8.8:15         8.8.8.8:15
icmp 203.0.113.5:16    192.168.10.51:16   8.8.8.8:16         8.8.8.8:16
icmp 203.0.113.5:17    192.168.10.51:17   8.8.8.8:17         8.8.8.8:17
icmp 203.0.113.5:24    192.168.10.51:24   8.8.8.8:24         8.8.8.8:24
icmp 203.0.113.5:25    192.168.10.51:25   8.8.8.8:25         8.8.8.8:25
icmp 203.0.113.5:26    192.168.10.51:26   8.8.8.8:26         8.8.8.8:26
```

Restore the primary path (`no shutdown` on Gi0/2):

```
R1#show ip route
Gateway of last resort is 203.0.113.2 to network 0.0.0.0

     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/1
L       10.0.0.1/32 is directly connected, GigabitEthernet0/1
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
     192.168.30.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.30.0/24 is directly connected, GigabitEthernet0/0.30
L       192.168.30.1/32 is directly connected, GigabitEthernet0/0.30
S    192.168.40.0/24 [1/0] via 10.0.0.2
     192.168.50.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.50.0/24 is directly connected, GigabitEthernet0/0.50
L       192.168.50.1/32 is directly connected, GigabitEthernet0/0.50
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/2
L       203.0.113.1/32 is directly connected, GigabitEthernet0/2
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   1 ms      0 ms      1 ms      8.8.8.8

Trace complete.
```

## Conclusion

- Configured: the fault (floating default removed, then Gi0/2 shut) and the fix (floating default re-added).
- Verified: the captures above. Before the fault, only the AD 1 default was installed. With the fault active, R1 had no gateway of last resort and PC-ADMIN's Internet pings failed, while internal, branch and VLAN 50 tests still passed. After the fix with Gi0/2 down, R1 installed the AD 10 default, traffic went via `10.0.0.2` (3 hops) and R2 translated to `203.0.113.5`. After restore, the AD 1 default returned (2 hops).
- Explained:
  - **Why nothing looked wrong at first:** a floating route is a static route with a higher AD, so it stays out of `show ip route` while the primary exists. Its absence only shows in the running config, and only matters when the primary fails.
  - **Why the unreachable reply came from `192.168.10.1`:** R1 had no route, so it answered the client itself.
  - **Why the branch and internal traffic kept working:** R2 has its own default via R3, and the HQ-branch and inter-VLAN routes do not depend on R1's default.
  - **Lesson:** a backup route is only proven by testing the failover. Check the configuration and shut the primary path, not just the routing table.