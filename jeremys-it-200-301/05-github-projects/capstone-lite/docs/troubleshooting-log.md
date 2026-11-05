# Troubleshooting Log — Capstone-Lite

Two kinds of entries here:

1. **Design errors caught before building** (topology diagram contradictions)
2. **Build/runtime issues** — add your own as you hit them

---

## 1. Design errors found in the original topology diagram

These were caught on paper before any device was configured. Documenting them
because they're the most instructive part: a diagram that can't be built is
worse than no diagram.

| # | Symptom in source diagram | Why it's impossible | Fix applied |
|---|---------------------------|---------------------|-------------|
| 1 | SW1 G0/1 shown connected to **both** R1 and SW2 | One physical port can't be in two links | SW1 G0/1 → R1, SW1 G0/2 → SW2 |
| 2 | Trunk B labeled as **Gig** on both ends | SW1's Gig ports are G0/1 and G0/2 — both already used; SW2 G0/2 was shown but never configured | Trunk B moved to **F0/23 ↔ F0/23** |
| 3 | Both inter-switch links drawn as Gig | Only one real Gig path exists between the switches | Trunk A = Gig (forwarding), Trunk B = Fast (blocked) |
| 4 | Text said "Gig preferred over Fast" but diagram showed two Gig links | STP cost comparison is meaningless if both links are the same speed | Now consistent: cost 4 vs 19 → G0/1 forwards, F0/23 blocks |
| 5 | SW2 G0/2 drawn but unused in every config | Diagram and configs disagree — a reader can't tell if it's spare or a mistake | Removed from diagram; documented as free |

**Lesson:** verify port inventory against the device model before drawing.
A 2960 has exactly two Gig ports; using one twice invalidates every downstream
STP expectation.

---

## 2. Runtime issues

### Template — copy this for each issue

```
### Issue: <one-line symptom>

**Where:** <device / PC / link>
**When:** <which step>
**Symptom:** <what you observed>
**Command run:** <show command>
**Output showed:** <key line>
**Root cause:** <why>
**Fix:** <what you changed>
**Verified by:** <command / ping that now passes>
```

---

### Issue: Subinterfaces down/down on R1

**Where:** R1
**When:** Step 6, immediately after configuring G0/0.10/.20/.30
**Symptom:** `show ip interface brief` showed G0/0.10/.20/.30 as `down/down`
**Command run:** `show ip interface brief`
**Root cause:** Two candidates — physical G0/0 still shut down, or SW1 G0/1 not
actually trunking (a subinterface only comes up if the parent is up *and* the
switch is passing tagged frames)
**Fix:** `no shutdown` on R1 G0/0, then confirmed `show interfaces trunk` on SW1
showed G0/1 trunking with VLANs 10,20,30 allowed
**Verified by:** `show ip interface brief` → all three subinterfaces up/up;
`show ip route` → three connected routes

*Note: this was a known failure mode called out in the build guide, not a
surprise. Recording it because the diagnostic path is the reusable part.*

---

### Issue: Same-VLAN hosts on different switches can't ping

**Where:** PC1 ↔ PC4 (both VLAN 10)
**Symptom:** ping fails; PC1 can ping its gateway fine
**Command run:** `show interfaces trunk` on SW1 and SW2
**Root cause:** Trunk up but VLAN 10 missing from the allowed list on one side
**Fix:** corrected `switchport trunk allowed vlan 10,20,30` on both ends
**Verified by:** `show interfaces trunk` → VLAN 10 active in the allowed and
forwarding lists; ping PC1 → PC4 succeeds
**Lesson:** an interface can report "trunking" and still not carry your VLAN.
Always check the *allowed and active* columns, not just the mode.

---

### Issue: F0/23 unexpectedly forwarding after failover test

**Where:** SW2 F0/23
**When:** After reconnecting the Gig link in Step 9
**Symptom:** F0/23 still Forwarding ~10 s after the cable was reinserted
**Root cause:** Not a fault — PVST reconvergence takes up to ~50 s. The Gig link
does not reclaim the root port instantly.
**Fix:** none required; waited for convergence
**Verified by:** `show spanning-tree vlan 10` → G0/1 Root/FWD, F0/23 Altn/BLK
**Lesson:** fast link ≠ instant reclaim under classic PVST. This is the argument
for rapid-PVSTP in production.

---

### Issue: PC7 unreachable from campus VLANs

**Where:** R1 / R2
**When:** Step 10
**Symptom:** PC1 → PC7 fails; R1 has the route, R2 looks fine
**Command run:** `show ip route` on R2
**Root cause:** Return path missing — R2 had no route back to 192.168.10.0/24, so
the echo reply was dropped
**Fix:** added the three static routes on R2 pointing to 10.0.0.1
**Verified by:** `show ip route` on R2 → 3× `S`; PC1 → PC7 succeeds;
PC7 → PC1 succeeds
**Lesson:** one-directional ping failure almost always means the return route is
missing. Check the far side, not the near side.

---

## 3. Quick diagnostic reference

| Symptom | First command | Where |
|---|---|---|
| PC can't ping its own gateway | PC IP/mask/gateway + `show vlan brief` | PC + switch |
| Same-VLAN PCs fail across switches | `show interfaces trunk` | SW1 + SW2 |
| Different-VLAN PCs fail | `show ip interface brief` | R1 |
| Trunk missing or not-trunking | `show interfaces trunk` | Both switches |
| Unexpected STP blocking | `show spanning-tree vlan 10` | Both switches |
| PC7 unreachable from campus | `show ip route` | R1 |
| PC7 can't return traffic | `show ip route` | R2 |
| Subinterface down on R1 | `show interfaces` | R1 |