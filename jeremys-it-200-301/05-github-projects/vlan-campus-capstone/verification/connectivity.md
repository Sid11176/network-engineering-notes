# Connectivity Verification (Session 3)

Network: `v1-core.pkt`. Gate item: HQ clients reach the simulated `8.8.8.8` destination, and R1's route table matches the design.

## Status

| Test | Status | Evidence |
| --- | --- | --- |
| PC-ADMIN, PC-USERS, PC-IT ping 8.8.8.8 | Captured: 4/4 replies, 0% loss on each | Section 1 |
| R1 route table | Captured | Section 2 |
| Each HQ PC pings its own VLAN gateway | Captured: 4/4 replies, 0% loss on each | Section 3 |
| HQ PCs ping PC-BRANCH (Session 2 still passes) | Captured: 3 runs, each 4/4 replies, 0% loss | Section 4 |

## 1. Ping to the simulated ISP loopback

### PC-ADMIN

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

### PC-USERS

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

### PC-IT

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

All three clients get 4/4 replies. The reply TTL of 254 is consistent with the reply passing through one router (R1) on its way back from R3.

A reply from `8.8.8.8` proves reachability to the simulated loopback on R3 only. It does not prove real Internet access.

## 2. R1 route table

### R1: `show ip route`

```
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
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/2
L       203.0.113.1/32 is directly connected, GigabitEthernet0/2
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

- `S*  0.0.0.0/0 via 203.0.113.2` is the primary default route toward R3.
- `S  192.168.40.0/24 via 10.0.0.2` is the Session 2 route to the branch, still present.
- `C`/`L` entries exist for Gi0/0.10, Gi0/0.20, Gi0/0.30, the R1–R2 transit link (Gi0/1), and the ISP link (Gi0/2).

## 3. Each HQ PC to its own VLAN gateway

### PC-ADMIN

```
C:\>ping 192.168.10.1

Pinging 192.168.10.1 with 32 bytes of data:

Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=1ms TTL=255
Reply from 192.168.10.1: bytes=32 time<1ms TTL=255
Reply from 192.168.10.1: bytes=32 time=1ms TTL=255

Ping statistics for 192.168.10.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms
```

### PC-USERS

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
```

### PC-IT

```
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

All three clients reach their own gateway with 4/4 replies. The reply TTL of 255 shows the reply came directly from R1's subinterface with no router in between, so DHCP-assigned addressing, the access ports, the trunk, and the ROAS subinterfaces all work.

## 4. HQ PCs to PC-BRANCH (Session 2 regression)

Three runs of `ping 192.168.40.10` from the HQ PCs.

```
C:\>ping 192.168.40.10

Pinging 192.168.40.10 with 32 bytes of data:

Reply from 192.168.40.10: bytes=32 time=1ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.40.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 192.168.40.10

Pinging 192.168.40.10 with 32 bytes of data:

Reply from 192.168.40.10: bytes=32 time<1ms TTL=126
Reply from 192.168.40.10: bytes=32 time=10ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.40.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 2ms

C:\>ping 192.168.40.10

Pinging 192.168.40.10 with 32 bytes of data:

Reply from 192.168.40.10: bytes=32 time<1ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126
Reply from 192.168.40.10: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.40.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

All three runs get 4/4 replies. The reply TTL of 126 is consistent with the reply crossing two routers (R2, then R1), so the Session 2 path over the R1–R2 link still works after adding DHCP, the default route, and NAT. The `10 ms` reply in run 2 is a first-packet delay and not a loss.

## Conclusion

- Configured: default route, DHCP clients, and NAT are in place.
- Verified: HQ clients get valid gateways, reach their own gateway, reach `8.8.8.8`, and still reach PC-BRANCH.
- Explained:
  - **What a reply from `8.8.8.8` proves:** the full round trip works. The client's DHCP settings and gateway are correct, R1 has a working default route to R3, NAT/PAT translates the private source, R3 is reachable on the `203.0.113.0/30` link, and the reply comes back and is translated to the right client.
  - **What it does not prove:** `8.8.8.8` is a loopback on R3, so this is not a real Internet connection. It does not prove public DNS resolution, access to external websites, the health of anything beyond R3, or that any other protocol or port works. It is also a single ICMP test and says nothing about TCP or UDP applications.













# Routing and Connectivity Verification (Session 5)

Network: `v2-standout.pkt`. Change this session: VLAN 50 (192.168.50.0/24, gateway R1 `Gi0/0.50`) and a route to it on R2 via 10.0.0.1.

## Evidence

1. **R1 routing table.** Run `show ip route`.
   Expected: connected 192.168.10/20/30/50.0/24, a static route to 192.168.40.0/24 via 10.0.0.2, and the default route via 203.0.113.2.

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

2. **R2 routing table.** Run `show ip route`.
   Expected: static routes to 192.168.10/20/30/50.0/24 via 10.0.0.1 and the default route via 203.0.113.6.

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

3. **Branch reaches HQ services.** From PC-BRANCH, ping 192.168.50.10 and 192.168.50.11. From PC-ADMIN, ping PC-BRANCH.
   Expected: all succeed. The first ping may time out while ARP resolves.

```
PC-BRANCH
C:\>ping 192.168.50.10

Pinging 192.168.50.10 with 32 bytes of data:

Reply from 192.168.50.10: bytes=32 time<1ms TTL=126
Reply from 192.168.50.10: bytes=32 time<1ms TTL=126
Reply from 192.168.50.10: bytes=32 time<1ms TTL=126
Reply from 192.168.50.10: bytes=32 time=1ms TTL=126

Ping statistics for 192.168.50.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 1ms, Average = 0ms

C:\>ping 192.168.50.11

Pinging 192.168.50.11 with 32 bytes of data:

Reply from 192.168.50.11: bytes=32 time<1ms TTL=126
Reply from 192.168.50.11: bytes=32 time=1ms TTL=126
Reply from 192.168.50.11: bytes=32 time=17ms TTL=126
Reply from 192.168.50.11: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.50.11:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 17ms, Average = 4ms

PC-ADMIN
C:\>ping 192.168.40.51

Pinging 192.168.40.51 with 32 bytes of data:

Reply from 192.168.40.51: bytes=32 time<1ms TTL=126
Reply from 192.168.40.51: bytes=32 time=10ms TTL=126
Reply from 192.168.40.51: bytes=32 time=10ms TTL=126
Reply from 192.168.40.51: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.40.51:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 10ms, Average = 5ms

```

4. **Normal WAN path.** From PC-ADMIN, run `tracert 8.8.8.8`.
   Expected: 192.168.10.1, then 8.8.8.8 (R3 answers for its loopback).

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   0 ms      0 ms      0 ms      8.8.8.8

Trace complete.
```

## Conclusion

- Configured: the VLAN 50 gateway on R1 and the VLAN 50 route on R2.
- Verified: the four captures above.
- Explained: the branch can reach the server VLAN because R2 sends 192.168.50.0/24 to R1, and R1's `Gi0/0.50` is the gateway for that VLAN.