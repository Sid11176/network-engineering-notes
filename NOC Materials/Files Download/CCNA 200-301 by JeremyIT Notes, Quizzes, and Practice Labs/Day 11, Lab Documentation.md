# 📝 Lab Documentation: DTP & VTP Configuration

### 🗺️ Topology & Interface Mapping
*   **SW1:** F0/1 (PC1-VLAN10), F0/2 (PC2-VLAN10), F0/3 (PC8-VLAN20), G0/1 (Trunk to SW2)
*   **SW2:** G0/1 (Trunk to SW1), G0/2 (Trunk to SW3), F0/1 (PC3-VLAN40), F0/2 (PC4-VLAN40)
*   **SW3:** G0/1 (Trunk to SW2), F0/1 (PC5-VLAN10), F0/2 (PC6-VLAN30), F0/3 (PC7-VLAN30), F0/4 (PC9-VLAN20)

---

### Step 1: Configure Trunks & Disable DTP
**Goal:** Establish inter-switch links as trunks and stop DTP negotiation frames.

**Commands:**
```text
! On SW1
SW1(config)# interface g0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport nonegotiate

! On SW2
SW2(config)# interface range g0/1 - 2
SW2(config-if-range)# switchport mode trunk
SW2(config-if-range)# switchport nonegotiate

! On SW3
SW3(config)# interface g0/1
SW3(config-if)# switchport mode trunk
SW3(config-if)# switchport nonegotiate
```

**Verification:**
Run `show interfaces switchport` on each trunk port.
*   **Administrative Mode:** trunk
*   **Operational Mode:** trunk
*   **Negotiation of Trunking:** Off (DTP is disabled)

---

### Step 2: Configure VTP Server (SW1)
**Goal:** Set SW1 as the central VTP server and create VLANs.

**Commands:**
```text
SW1(config)# vtp domain CCNA
SW1(config)# vlan 10
SW1(config-vlan)# vlan 20
SW1(config-vlan)# vlan 30
```

**Lab Question:** Have SW2 and SW3 added VLANs 10, 20, and 30?
**Answer:** **Yes.** Because SW2 and SW3 are in their default VTP Server mode and have no domain name configured, they automatically join the "CCNA" domain upon receiving SW1's advertisement and sync their databases to match SW1's higher revision number.

---

### Step 3: Configure VTP Transparent (SW2)
**Goal:** Stop SW2 from syncing its database while still allowing it to forward VTP advertisements.

**Commands:**
```text
SW2(config)# vtp mode transparent
SW2(config)# vlan 40
```

**Lab Question:** Is VLAN40 added to the VLAN database of SW1/SW3?
**Answer:** **No.** A switch in Transparent mode does not advertise its own VLAN database to other switches. It will only forward advertisements it receives from SW1.

---

### Step 4: Configure VTP Client (SW3)
**Goal:** Set SW3 to client mode so it cannot modify the VLAN database.

**Commands:**
```text
SW3(config)# vtp mode client
SW3(config)# vlan 50
```

**Lab Question:** Try to configure VLAN50 on SW3. Is it added?
**Answer:** **No.** The command will be rejected. VTP Clients cannot add, modify, or delete VLANs in the CLI.

---

### Step 5: Configure Access Ports
**Goal:** Assign the host ports to their correct VLANs based on the topology.

**Commands:**
```text
! On SW1
SW1(config)# interface range f0/1 - 2
SW1(config-if-range)# switchport mode access
SW1(config-if-range)# switchport access vlan 10
SW1(config)# interface f0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20

! On SW2
SW2(config)# interface range f0/1 - 2
SW2(config-if-range)# switchport mode access
SW2(config-if-range)# switchport access vlan 40

! On SW3
SW3(config)# interface f0/1
SW3(config-if)# switchport mode access
SW3(config-if)# switchport access vlan 10
SW3(config)# interface range f0/2 - 3
SW3(config-if-range)# switchport mode access
SW3(config-if-range)# switchport access vlan 30
SW3(config)# interface f0/4
SW3(config-if)# switchport mode access
SW3(config-if)# switchport access vlan 20
```

**Lab Question:** Is DTP still enabled on the switchports?
**Answer:** **No.** Configuring a port with `switchport mode access` automatically disables DTP negotiation (Negotiation of Trunking will show as **Off**). The port will no longer send DTP frames.

*(Note: Verify end-to-end connectivity within the same VLANs, for example, PC1 pinging PC5 since they are both in VLAN 10).*