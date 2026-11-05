# IP Plan — Capstone-Lite

## VLANs

| VLAN ID | Name  | Subnet           | Gateway        | Purpose              |
|---------|-------|------------------|----------------|----------------------|
| 10      | ADMIN | 192.168.10.0/24  | 192.168.10.1   | Administrative hosts |
| 20      | USERS | 192.168.20.0/24  | 192.168.20.1   | General users        |
| 30      | IT    | 192.168.30.0/24  | 192.168.30.1   | IT staff             |

All gateways are subinterfaces on R1 G0/0 (router-on-a-stick).

## Point-to-Point Transit

| Link      | Network       | Mask            | R1 side     | R2 side     |
|-----------|---------------|-----------------|-------------|-------------|
| R1 ↔ R2   | 10.0.0.0/30   | 255.255.255.252 | 10.0.0.1    | 10.0.0.2    |

A /30 is used because the link only ever needs two usable addresses.

## Remote LAN

| Network           | Gateway       | Location |
|-------------------|---------------|----------|
| 192.168.40.0/24   | 192.168.40.1  | R2 G0/1  |

## Router Interfaces

| Device | Interface | Address          | Description        |
|--------|-----------|------------------|--------------------|
| R1     | G0/0      | no IP            | Trunk to SW1       |
| R1     | G0/0.10   | 192.168.10.1/24  | VLAN 10 gateway    |
| R1     | G0/0.20   | 192.168.20.1/24  | VLAN 20 gateway    |
| R1     | G0/0.30   | 192.168.30.1/24  | VLAN 30 gateway    |
| R1     | G0/1      | 10.0.0.1/30      | Link to R2         |
| R2     | G0/0      | 10.0.0.2/30      | Link to R1         |
| R2     | G0/1      | 192.168.40.1/24  | Remote LAN         |

## Switch Port Assignments

| Device | Port  | Mode   | VLAN / Allowed VLANs | Description        |
|--------|-------|--------|----------------------|--------------------|
| SW1    | F0/1  | access | 10                   | PC1_ADMIN          |
| SW1    | F0/2  | access | 20                   | PC2_USERS          |
| SW1    | F0/3  | access | 30                   | PC3_IT             |
| SW1    | G0/1  | trunk  | 10,20,30             | TRUNK_TO_R1        |
| SW1    | G0/2  | trunk  | 10,20,30             | TRUNK_TO_SW2 (Gig) |
| SW1    | F0/23 | trunk  | 10,20,30             | REDUNDANT (Fast)   |
| SW2    | F0/1  | access | 10                   | PC4_ADMIN          |
| SW2    | F0/2  | access | 20                   | PC5_USERS          |
| SW2    | F0/3  | access | 30                   | PC6_IT             |
| SW2    | G0/1  | trunk  | 10,20,30             | TRUNK_TO_SW1 (Gig) |
| SW2    | F0/23 | trunk  | 10,20,30             | REDUNDANT (Fast)   |
| SW2    | G0/2  | —      | unused               | Free               |

## End Devices

| Host | IP            | Mask          | Gateway       | VLAN | Switch port |
|------|---------------|---------------|---------------|------|-------------|
| PC1  | 192.168.10.10 | 255.255.255.0 | 192.168.10.1  | 10   | SW1 F0/1    |
| PC2  | 192.168.20.10 | 255.255.255.0 | 192.168.20.1  | 20   | SW1 F0/2    |
| PC3  | 192.168.30.10 | 255.255.255.0 | 192.168.30.1  | 30   | SW1 F0/3    |
| PC4  | 192.168.10.11 | 255.255.255.0 | 192.168.10.1  | 10   | SW2 F0/1    |
| PC5  | 192.168.20.11 | 255.255.255.0 | 192.168.20.1  | 20   | SW2 F0/2    |
| PC6  | 192.168.30.11 | 255.255.255.0 | 192.168.30.1  | 30   | SW2 F0/3    |
| PC7  | 192.168.40.10 | 255.255.255.0 | 192.168.40.1  | —    | R2 G0/1     |

## Static Routes

| Device | Destination      | Next Hop  | Purpose                        |
|--------|------------------|-----------|--------------------------------|
| R1     | 192.168.40.0/24  | 10.0.0.2  | Reach the remote LAN           |
| R2     | 192.168.10.0/24  | 10.0.0.1  | Return traffic to VLAN 10      |
| R2     | 192.168.20.0/24  | 10.0.0.1  | Return traffic to VLAN 20      |
| R2     | 192.168.30.0/24  | 10.0.0.1  | Return traffic to VLAN 30      |