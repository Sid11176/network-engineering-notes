# DHCP Verification (Session 3)

Network: `v1-core.pkt`. DHCP server: R1. Pools: `VLAN10-ADMIN`, `VLAN20-USERS`, `VLAN30-IT` (`.1`–`.50` excluded).

## Result summary

| Client | IPv4 address | Mask | Default gateway | DHCP server |
| --- | --- | --- | --- | --- |
| PC-ADMIN | 192.168.10.51 | 255.255.255.0 | 192.168.10.1 | 192.168.10.1 |
| PC-USERS | 192.168.20.51 | 255.255.255.0 | 192.168.20.1 | 192.168.20.1 |
| PC-IT | 192.168.30.51 | 255.255.255.0 | 192.168.30.1 | 192.168.30.1 |

Each client received the first address after the excluded block, the correct mask, and its own VLAN gateway.

## Client output (`ipconfig /all`)

Only the FastEthernet0 section is kept. The unused Bluetooth adapter section was removed.

### PC-ADMIN

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0001.42C4.85AA
   Link-local IPv6 Address.........: FE80::201:42FF:FEC4:85AA
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.10.51
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.10.1
   DHCP Servers....................: 192.168.10.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-64-E8-A0-B6-00-01-42-C4-85-AA
   DNS Servers.....................: ::
                                     0.0.0.0
```

### PC-USERS

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0000.0C76.2C94
   Link-local IPv6 Address.........: FE80::200:CFF:FE76:2C94
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.20.51
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.20.1
   DHCP Servers....................: 192.168.20.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-0D-EB-65-08-00-00-0C-76-2C-94
   DNS Servers.....................: ::
                                     0.0.0.0
```

### PC-IT

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 0009.7C61.A3E1
   Link-local IPv6 Address.........: FE80::209:7CFF:FE61:A3E1
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.30.51
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.30.1
   DHCP Servers....................: 192.168.30.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-84-40-E9-01-00-09-7C-61-A3-E1
   DNS Servers.....................: ::
                                     0.0.0.0
```

## R1 lease table

### R1: `show ip dhcp binding`

```
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.10.51    0001.42C4.85AA           --                     Automatic
192.168.20.51    0000.0C76.2C94           --                     Automatic
192.168.30.51    0009.7C61.A3E1           --                     Automatic
```

R1 shows one automatic binding per HQ PC, and the MAC addresses match the clients above.

DNS servers show `0.0.0.0` on the clients. This is expected because no `dns-server` option is configured until Session 5.

## Conclusion

- Configured: three pools with excluded ranges and the correct default router.
- Verified: all three HQ PCs received valid leases (above). The gateway ping test is in `connectivity.md`.
- Explained:
  - DHCP gives a client its network settings automatically instead of by hand: an IP address, a subnet mask, a default gateway, and a DNS server when one is configured.
  - The client broadcasts a request, and R1 receives it on the ROAS subinterface for that VLAN. R1 answers from the pool that matches that subnet, which is how each VLAN gets an address in its own range.
  - The `.1`–`.50` block is excluded, so leases start at `.51`. Gateways and fixed addresses stay outside the lease range and cannot collide with a client.
  - The DNS server shows `0.0.0.0` because no `dns-server` option exists yet. It is added in Session 5.