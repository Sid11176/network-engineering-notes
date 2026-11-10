# Failover Verification – Session 4

- **Date:** 2026-11-10
- **File:** `packet-tracer/v1-core.pkt`
- **Scope:** Branch DHCP from R2, R1 primary default route via R3, floating default route via R2, R2 PAT for the backup path
- **Gate being verified:**
  - Branch PC receives a valid DHCP address from R2
  - HQ and branch clients reach each other in both directions
  - Branch client reaches the simulated ISP loopback (`8.8.8.8`)
  - R1 uses the direct R3 route while it is available
  - With R1–R3 down, the floating default via R2 becomes active and HQ still reaches `8.8.8.8`
  - With the link restored, the preferred route returns

Addresses used: PC-ADMIN `192.168.10.51` (DHCP), PC-BRANCH `192.168.40.51` (DHCP).

---

## 0. Branch DHCP

**PC-BRANCH – `ipconfig /all` (FastEthernet0 adapter):**

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0060.5C23.7B88
   Link-local IPv6 Address.........: FE80::260:5CFF:FE23:7B88
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.40.51
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.40.1
   DHCP Servers....................: 192.168.40.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-B1-DD-92-B7-00-60-5C-23-7B-88
   DNS Servers.....................: ::
                                     0.0.0.0
```

**Observation:** PC-BRANCH received `192.168.40.51/24` with gateway `192.168.40.1`. The DHCP server is `192.168.40.1`, which is R2's branch LAN interface. The DNS server shows `0.0.0.0` because no DNS option is configured yet (planned for Session 5). **PASS**

---

## 1. Normal state (primary path)

**R1 – `show ip route static`:**

```
R1#show ip route static 
S    192.168.40.0/24 [1/0] via 10.0.0.2
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

**R1 – `show running-config | include ip route`:**

```
R1#show running-config | include ip route
ip route 192.168.40.0 255.255.255.0 10.0.0.2 
ip route 0.0.0.0 0.0.0.0 203.0.113.2 
ip route 0.0.0.0 0.0.0.0 10.0.0.2 10
```

**PC-ADMIN – `ping 8.8.8.8` and `tracert 8.8.8.8`:**

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

C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      1 ms      192.168.10.1
  2   0 ms      0 ms      0 ms      8.8.8.8

Trace complete.
```

**PC-BRANCH – `ping 8.8.8.8`:**

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

**HQ ↔ Branch – PC-ADMIN → PC-BRANCH:**

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

**HQ ↔ Branch – PC-BRANCH → PC-ADMIN:**

```
C:\>ping 192.168.10.51

Pinging 192.168.10.51 with 32 bytes of data:

Reply from 192.168.10.51: bytes=32 time<1ms TTL=126
Reply from 192.168.10.51: bytes=32 time<1ms TTL=126
Reply from 192.168.10.51: bytes=32 time<1ms TTL=126
Reply from 192.168.10.51: bytes=32 time<1ms TTL=126

Ping statistics for 192.168.10.51:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

**Observation:** R1's default route is the primary one via R3 (`203.0.113.2`, AD 1). The floating route via `10.0.0.2` (AD 10) is configured but not installed in the routing table. The trace from PC-ADMIN reaches `8.8.8.8` directly after R1. PC-BRANCH reaches `8.8.8.8`. HQ and branch reach each other in both directions. **PASS**

---

## 2. Failover – R1 Gi0/2 shut down

**Commands and link state:**

```
R1(config)#interface gigabitEthernet 0/2
R1(config-if)#shutdown

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/2, changed state to down
```

**R1 – `show ip route static`:**

```
R1#show ip route static 
S    192.168.40.0/24 [1/0] via 10.0.0.2
S*   0.0.0.0/0 [10/0] via 10.0.0.2
```

**PC-ADMIN – `ping 8.8.8.8` and `tracert 8.8.8.8`:**

```
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time<1ms TTL=253
Reply from 8.8.8.8: bytes=32 time<1ms TTL=253
Reply from 8.8.8.8: bytes=32 time<1ms TTL=253
Reply from 8.8.8.8: bytes=32 time<1ms TTL=253

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   0 ms      0 ms      0 ms      10.0.0.2
  3   0 ms      0 ms      0 ms      8.8.8.8

Trace complete.
```

**R2 – `show ip nat translations`:**

```
R2#show ip nat translations 
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.5:204   192.168.10.51:204  8.8.8.8:204        8.8.8.8:204
icmp 203.0.113.5:205   192.168.10.51:205  8.8.8.8:205        8.8.8.8:205
icmp 203.0.113.5:206   192.168.10.51:206  8.8.8.8:206        8.8.8.8:206
icmp 203.0.113.5:207   192.168.10.51:207  8.8.8.8:207        8.8.8.8:207
icmp 203.0.113.5:214   192.168.10.51:214  8.8.8.8:214        8.8.8.8:214
icmp 203.0.113.5:215   192.168.10.51:215  8.8.8.8:215        8.8.8.8:215
icmp 203.0.113.5:216   192.168.10.51:216  8.8.8.8:216        8.8.8.8:216
```

**Observation:** The console messages confirm Gi0/2 went administratively down. R1's default route changed to the floating route `[10/0] via 10.0.0.2`. The trace from PC-ADMIN gained a hop through R2 (`10.0.0.2`) before reaching `8.8.8.8`, and TTL dropped from 254 to 253. R2 translated HQ address `192.168.10.51` to its own outside address `203.0.113.5`, so the backup path works through R2's PAT. **PASS**

---

## 3. Restore – R1 Gi0/2 brought back up

**Command:**

```
R1(config)#interface gigabitEthernet 0/2
R1(config-if)#no shutdown
```

**R1 – `show ip route static`:**

```
R1#show ip route static 
S    192.168.40.0/24 [1/0] via 10.0.0.2
S*   0.0.0.0/0 [1/0] via 203.0.113.2
```

**PC-ADMIN – `tracert 8.8.8.8`:**

```
C:\>tracert 8.8.8.8

Tracing route to 8.8.8.8 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      192.168.10.1
  2   *         6 ms      0 ms      8.8.8.8

Trace complete.
```

**Observation:** The primary default route `[1/0] via 203.0.113.2` returned and the trace is back to the direct two-hop path. The single `*` on hop 2 is a first-packet timeout while the route re-converged; the later replies succeeded. **PASS**

---

## 4. Limitation

The floating static route has no IP SLA or tracking. It detects a failed
R1–R3 link (Gi0/2 going down) but not an ISP outage beyond R3 while Gi0/2
stays up; in that case R1 would keep using its primary default route.
This test proves failover for a directly detectable link failure, not
end-to-end Internet health monitoring.

---

## 5. Explain it back

**What is a floating static route?**

A floating static route is a backup static route configured with a higher
administrative distance than the primary route. It stays out of the routing
table while the primary route is available, and it is installed only when the
primary route is removed (for example, when the R1–R3 link goes down).

**Why does a higher administrative distance make a static route less preferred?**

Administrative distance is a trust rating for the source of a route, not a
physical or time distance. When a router has more than one route to the same
destination from different sources, it installs the one with the lowest AD. In
this lab the primary default route has AD 1 and the floating route has AD 10,
so R1 installs the AD 1 route and keeps the AD 10 route out of the table until
the primary is gone.

**What conditions must be true for HQ traffic to use R2 and then reach the ISP?**

- The primary default route via R3 has been removed from R1's routing table,
  for example because the R1–R3 link is down.
- R1 has the floating default route via 10.0.0.2 with AD 10.
- The R1–R2 transit link is up.
- R2 has a default route to R3 (203.0.113.6) and its R2–R3 link is up.
- R2's NAT is set up to translate the HQ subnets (they are in its NAT match
  ACL, with Gi0/2 as the outside interface).

**Why is NAT configuration on R2 necessary for the backup path?**

HQ traffic reaching R2 still has private source addresses (192.168.x.x). The
R2–R3 link is the outside, and R3 has no routes to private subnets, so replies
couldn't return. R2 translates the HQ source addresses to its own outside
address 203.0.113.5, as shown in the NAT translation table, so R3 can reply
to a reachable address.

---

## 6. Gate summary

- [x] Branch PC receives a valid DHCP address from R2
- [x] HQ and branch clients reach each other in both directions
- [x] Branch client reaches the simulated ISP loopback `8.8.8.8`
- [x] R1 uses the direct R3 route while it is available
- [x] With R1–R3 down, the floating default via R2 is active and HQ reaches `8.8.8.8`
- [x] With the link restored, the preferred route returns