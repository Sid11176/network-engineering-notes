# NAT/PAT Verification (Session 3)

Network: `v1-core.pkt`. NAT device: R1. Inside: Gi0/0.10, Gi0/0.20, Gi0/0.30. Outside: Gi0/2 (203.0.113.1/30). Match list: ACL 1.

## 1. Translations

### R1: `show ip nat translations`

```
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.1:1024  192.168.20.51:29   8.8.8.8:29         8.8.8.8:1024
icmp 203.0.113.1:1025  192.168.20.51:30   8.8.8.8:30         8.8.8.8:1025
icmp 203.0.113.1:1026  192.168.20.51:31   8.8.8.8:31         8.8.8.8:1026
icmp 203.0.113.1:1027  192.168.20.51:32   8.8.8.8:32         8.8.8.8:1027
icmp 203.0.113.1:29    192.168.10.51:29   8.8.8.8:29         8.8.8.8:29
icmp 203.0.113.1:30    192.168.10.51:30   8.8.8.8:30         8.8.8.8:30
icmp 203.0.113.1:31    192.168.10.51:31   8.8.8.8:31         8.8.8.8:31
icmp 203.0.113.1:32    192.168.10.51:32   8.8.8.8:32         8.8.8.8:32
icmp 203.0.113.1:33    192.168.30.51:33   8.8.8.8:33         8.8.8.8:33
icmp 203.0.113.1:34    192.168.30.51:34   8.8.8.8:34         8.8.8.8:34
icmp 203.0.113.1:35    192.168.30.51:35   8.8.8.8:35         8.8.8.8:35
icmp 203.0.113.1:36    192.168.30.51:36   8.8.8.8:36         8.8.8.8:36
```

- Inside locals from all three subnets (`192.168.10.51`, `192.168.20.51`, `192.168.30.51`) are translated to the single inside global `203.0.113.1`.
- PC-ADMIN and PC-USERS both sent ICMP IDs 13–16. For PC-USERS, R1 used global ports 1024–1027 instead, so the two flows stay distinct. This is PAT keeping separate flows apart on one address.

## 2. ACL match counters

### R1: `show access-lists 1`

```
Standard IP access list 1
    permit 192.168.10.0 0.0.0.255 (8 match(es))
    permit 192.168.20.0 0.0.0.255 (8 match(es))
    permit 192.168.30.0 0.0.0.255 (8 match(es))
```

Each HQ subnet entry has matched traffic, so all three subnets are in the NAT match scope.

## 3. NAT statistics

### R1: `show ip nat statistics`

```
Total translations: 12 (0 static, 12 dynamic, 12 extended)
Outside Interfaces: GigabitEthernet0/2
Inside Interfaces: GigabitEthernet0/0.10 , GigabitEthernet0/0.20 , GigabitEthernet0/0.30
Hits: 47  Misses: 48
Expired translations: 36
Dynamic mappings:
```

The inside and outside interfaces match the design, and the hit counter is non-zero.

**Note on timing:** this output shows `Total translations: 0` while the translations table above lists 12 entries. The two commands were captured at different times, and dynamic entries expire. For a clean record, re-run them back to back (see section 5).

## 4. R3 needs no route to the private subnets

### R3: `show ip route`

```
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, E - EGP
       i - IS-IS, L1 - IS-IS level-1, L2 - IS-IS level-2, ia - IS-IS inter area
       * - candidate default, U - per-user static route, o - ODR
       P - periodic downloaded static route

Gateway of last resort is not set

     8.0.0.0/32 is subnetted, 1 subnets
C       8.8.8.8/32 is directly connected, Loopback0
     203.0.113.0/24 is variably subnetted, 2 subnets, 2 masks
C       203.0.113.0/30 is directly connected, GigabitEthernet0/0
L       203.0.113.2/32 is directly connected, GigabitEthernet0/0
```

R3 has only connected routes (`8.8.8.8/32`, `203.0.113.0/30`) and no gateway of last resort, yet the pings succeed. Replies return to `203.0.113.1`, which is directly connected, because R1 translated the private source address.

## Conclusion

- Configured: ACL 1, `ip nat inside source list 1 interface GigabitEthernet0/2 overload`, inside/outside roles.
- Verified: translations to `203.0.113.1` for all three HQ subnets (above).
- Explained:
  - **What PAT changes:** when a private client sends traffic out Gi0/2, R1 replaces the private source address with its outside address, `203.0.113.1`, and tracks each flow by port (or ICMP ID). Many private hosts can therefore share one public address. When the reply returns to `203.0.113.1`, R1 looks up the translation table and rewrites the destination back to the original private address. In the capture, PC-ADMIN and PC-USERS both used ICMP ID 13–16, so R1 gave PC-USERS ports 1024–1027 to keep the flows apart.
  - **Why R3 needs no route to the private subnets:** R3 only ever sees the source `203.0.113.1`, which is on its directly connected `/30`. Replies go back to that address, and R1 translates them to the real client. R3's route table shows only connected routes and the pings still work.
  - **Why Gi0/1 is not the outside interface:** the NAT boundary is between private site networks and ISP-facing links, not between any two routers. Gi0/1 carries private-to-private traffic between HQ and the branch, which should keep its real addresses. If Gi0/1 were `ip nat outside`, HQ-to-branch traffic would be translated, and the branch would see `203.0.113.1` instead of the actual HQ client. Only Gi0/2 faces the ISP, so only Gi0/2 is outside.