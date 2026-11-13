# Troubleshooting Case 2: R1 default route removed

Network: `v2-standout.pkt` (restored clean copy). One change per fault.

This case has two stages because R1 has a floating backup default route via R2 (AD 10). Removing only the primary default does not break the Internet path, so Stage 1 shows a silent failover and Stage 2 removes the backup as well to produce the hard failure.

## Status

| Step | Status | Evidence |
| --- | --- | --- |
| Baseline | Captured | Section 1 |
| Stage 1: primary default removed | Captured | Section 2 |
| Stage 2: both defaults removed | Captured | Section 3 |
| Fix and recovery | Captured | Section 4 |

## Summary

| Item | Stage 1 (primary default removed) | Stage 2 (both defaults removed) |
| --- | --- | --- |
| **Symptom** | None visible to users. `8.8.8.8` still replies, but the path changes. | HQ clients cannot reach `8.8.8.8` or `http://publicwebsite.com`. Internal destinations still work. |
| **Scope** | HQ Internet traffic only (path change). | HQ Internet traffic only. VLANs 10, 20, 30, 50 and the branch are unaffected. PC-BRANCH still reaches `8.8.8.8` through R2's own default route. |
| **Layer** | Layer 3 (routing). | Layer 3 (routing). |
| **Root cause** | `ip route 0.0.0.0 0.0.0.0 203.0.113.2` removed from R1. The floating route via `10.0.0.2` (AD 10) is installed in its place. | Both default routes removed, so R1 has no gateway of last resort. |
| **Fix** | Re-add the primary default route. | Re-add both default routes. |
| **Verification** | `show ip route` shows the AD 1 default via `203.0.113.2`, and the path is back to one hop. | Same as Stage 1, plus the Internet tests pass again. |

## 1. Baseline

From PC-ADMIN: `ping 8.8.8.8`, `tracert 8.8.8.8`, `ping 192.168.40.51` (PC-BRANCH), `ping 192.168.50.10`.

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

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      1 ms      0 ms      192.168.10.1
  2   0 ms      0 ms      0 ms      8.8.8.8

Trace complete.
```

Expected: TTL 254, and `192.168.10.1` then `8.8.8.8`.

```
C:\>ping 192.168.40.51

Pinging 192.168.40.51 with 32 bytes of data:

Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.40.51:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
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

R1: `show ip route | include Gateway|0.0.0.0`

```
R1#show ip route | include 0.0.0.0
Gateway of last resort is 203.0.113.2 to network 0.0.0.0
     10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C       10.0.0.0/30 is directly connected, GigabitEthernet0/1
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

Expected: gateway of last resort `203.0.113.2`, and `S* 0.0.0.0/0 [1/0] via 203.0.113.2`.

## 2. Stage 1: primary default route removed

### Fault injected

R1:

```
R1(config)#no ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

```
R1#show running-config | include ip route
ip route 192.168.40.0 255.255.255.0 10.0.0.2 
ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
```

### Symptoms observed

| Test | Expected result | Observed |
| --- | --- | --- |
| PC-ADMIN `ping 8.8.8.8` | Still succeeds, TTL 253 | SUCCEED (TTL 253) |
| PC-ADMIN `tracert 8.8.8.8` | Three hops: `192.168.10.1`, `10.0.0.2`, `8.8.8.8` | SUCCEED (three hops, via `10.0.0.2`) |
| PC-ADMIN `ping 192.168.40.51` | Succeeds | SUCCEED |
| PC-ADMIN `ping 192.168.50.10` | Succeeds | SUCCEED |

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Request timed out.
Reply from 8.8.8.8: bytes=32 time=1ms TTL=253
Reply from 8.8.8.8: bytes=32 time<1ms TTL=253
Reply from 8.8.8.8: bytes=32 time=10ms TTL=253

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 3ms
```

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   0 ms      1 ms      0 ms      10.0.0.2
  3   10 ms     10 ms     0 ms      8.8.8.8

Trace complete.
```

### Diagnosis

1. **Scope.** Nothing fails, so there is no user-visible symptom. The only change is the path, shown by the extra hop and the lower TTL.
2. **Routing check.** R1 now uses the floating route.

```
R1#show ip route | include gateway | 0.0.0.0
Gateway of last resort is 10.0.0.2 to network 0.0.0.0
S*   0.0.0.0/0 [10/0] via 10.0.0.2
```

Expected: gateway of last resort `10.0.0.2`, and `S* 0.0.0.0/0 [10/0] via 10.0.0.2`.

3. **NAT check.** R2 translates the HQ client because the traffic leaves R2 Gi0/2. R2: `show ip nat translations`

```
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.5:48    192.168.10.51:48   8.8.8.8:48         8.8.8.8:48
icmp 203.0.113.5:49    192.168.10.51:49   8.8.8.8:49         8.8.8.8:49
icmp 203.0.113.5:50    192.168.10.51:50   8.8.8.8:50         8.8.8.8:50
icmp 203.0.113.5:51    192.168.10.51:51   8.8.8.8:51         8.8.8.8:51
```

Expected: inside local `192.168.10.51`, inside global `203.0.113.5`.

## 3. Stage 2: both default routes removed

### Fault injected

R1:

```
R1(config)#no ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
```

```
R1#show running-config | include ip route
ip route 192.168.40.0 255.255.255.0 10.0.0.2 
```

### Symptoms observed

| Test | Expected result | Observed |
| --- | --- | --- |
| PC-ADMIN `ping 8.8.8.8` | Fails (Destination host unreachable from `192.168.10.1`) | UNREACHABLE/FAIL |
| PC-ADMIN `ping 192.168.40.51` (internal) | Succeeds | SUCCEED |
| PC-ADMIN `ping 192.168.50.10` (internal) | Succeeds | SUCCEED |
| PC-ADMIN `nslookup publicwebsite.com` | Resolves to `198.51.100.10` | SUCCEED |
| PC-ADMIN `http://publicwebsite.com` | Does not load | SERVICE TIMEOUT/FAIL |
| PC-BRANCH `ping 8.8.8.8` | Still succeeds (R2 has its own default) | SUCCEED |

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.
Reply from 192.168.10.1: Destination host unreachable.
Request timed out.

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

```
C:\>ping 192.168.40.51

Pinging 192.168.40.51 with 32 bytes of data:

Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.40.51:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 192.168.50.10

Pinging 192.168.50.10 with 32 bytes of data:

Reply from 192.168.50.10: bytes=32 time<1ms TTL=127
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127
Reply from 192.168.50.10: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.50.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

```
C:\>nslookup publicwebsite.com

Server: [192.168.50.10]
Address:  192.168.50.10

Non-authoritative answer:
Name:   publicwebsite.com
Address:   198.51.100.10
```

![PC-ADMIN fails to open http://publicwebsite.com (Stage 2)](../evidence/screenshots/ts2-public-website-fail.png)

PC-BRANCH:

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

### Diagnosis

1. **Scope.** Only Internet-bound HQ traffic fails. Internal destinations and DNS resolution work, which separates a routing problem from a DNS or Layer 2 problem.
2. **Test separately.** The internal ping succeeds and the external ping fails, so the problem is the path out, not the HQ VLANs.
3. **Route table.** R1 has no gateway of last resort.

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
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/2
L       203.0.113.1/32 is directly connected, GigabitEthernet0/2
```

Expected: `Gateway of last resort is not set`, with no `S*` entry. The connected routes, the branch static route and the VLAN 50 route are still present.

4. **Interface check.** The links are fine, so the cause is not physical.

```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0     unassigned      YES NVRAM  up                    up 
GigabitEthernet0/0.10  192.168.10.1    YES manual up                    up 
GigabitEthernet0/0.20  192.168.20.1    YES manual up                    up 
GigabitEthernet0/0.30  192.168.30.1    YES manual up                    up 
GigabitEthernet0/0.50  192.168.50.1    YES manual up                    up 
GigabitEthernet0/1     10.0.0.1        YES NVRAM  up                    up 
GigabitEthernet0/2     203.0.113.1     YES NVRAM  up                    up 
Vlan1                  unassigned      YES unset  administratively down down
```

## 4. Fix and verification

R1:

```
R1(config)#ip route 0.0.0.0 0.0.0.0 203.0.113.2
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
```

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

Expected: gateway of last resort `203.0.113.2`, `S* 0.0.0.0/0 [1/0] via 203.0.113.2`. The AD 10 route is configured but not installed.

```
R1#show running-config | include ip route
ip route 192.168.40.0 255.255.255.0 10.0.0.2 
ip route 0.0.0.0 0.0.0.0 203.0.113.2 
ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
```

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254
Reply from 8.8.8.8: bytes=32 time=7ms TTL=254
Reply from 8.8.8.8: bytes=32 time<1ms TTL=254

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 7ms, Average = 1ms
```

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   0 ms      0 ms      0 ms      8.8.8.8

Trace complete.
```

Expected: TTL 254, and `192.168.10.1` then `8.8.8.8` (the primary path again).

![PC-ADMIN opens http://publicwebsite.com (after fix)](../evidence/screenshots/ts2-public-website-fixed.png)

## Conclusion

- Configured: the faults (primary default removed, then both defaults removed) and the fix (both defaults restored).
- Verified: the captures above.
- Explained:
  - **Why Stage 1 was invisible:** the floating route has a higher AD (10), so it is not installed while the primary exists. When the primary is removed, R1 installs it automatically. The Internet still works, but HQ traffic now goes R1 → R2 → R3, is translated on R2 to `203.0.113.5`, and the TTL drops from 254 to 253. A working ping does not prove the primary path is in use.
  - **Why Stage 2 breaks only external traffic:** R1 still has connected routes for every VLAN, the static route to the branch and the route to VLAN 50, so internal traffic needs no default route. Only destinations with no specific match, such as `8.8.8.8`, fall back to the default route, and R1 has none.
  - **Why `publicwebsite.com` resolves but does not load:** the DNS server is in VLAN 50 and still reachable, so it returns `198.51.100.10`. DNS only turns the name into an address. Routing then has to carry the packets, and that fails. This is the opposite of Case 1, where the DNS server itself was unreachable.
  - **Why PC-BRANCH is unaffected:** R2 has its own default route via `203.0.113.6` and its own NAT, so the branch does not depend on R1's default route.
  - **Lesson:** use `show ip route` and `tracert`, not only ping, to confirm which path traffic takes. The floating route has no tracking, so a failure of the primary is silent.