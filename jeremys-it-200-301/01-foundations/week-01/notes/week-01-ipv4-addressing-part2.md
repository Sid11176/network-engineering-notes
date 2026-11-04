# 🌐 IPv4 Addressing Part 2 — Notes

📅 **Date:** 10.25.2026

🎯 **Big Picture:** Learn how to calculate **Network / Broadcast / Usable IPs** from an IP address + prefix length, and how to configure IP addresses on Cisco router interfaces.

---

## 🧮 The “Big 5” Calculations Cheat Sheet

🔑 **Key idea:** To calculate subnet boundaries, look at the **host portion** — the bits after the prefix length.

Let **h = number of host bits**.

| # | Calculation | Rule |
| --- | --- | --- |
| 1 | 🖥️ **Maximum Hosts** | `2^h - 2` |
| 2 | 🏠 **Network Address** | Set all host bits to `0` |
| 3 | 📢 **Broadcast Address** | Set all host bits to `1` |
| 4 | ✅ **First Usable Address** | Network Address + 1 |
| 5 | ✅ **Last Usable Address** | Broadcast Address − 1 |

> ❗ Subtract **2** from max hosts because the **Network Address** and **Broadcast Address** cannot be assigned to hosts.
> 

### 📌 Common Host-Bit Examples

| Prefix | Host Bits | Max Hosts |
| --- | --- | --- |
| `/8` | 24 | `2²⁴ − 2 = 16,777,214` |
| `/16` | 16 | `2¹⁶ − 2 = 65,534` |
| `/24` | 8 | `2⁸ − 2 = 254` |

---

## 📝 Practice Examples

| Device | IP / Prefix | Class | Network Address | Broadcast Address | First Usable | Last Usable | Max Hosts |
| --- | --- | --- | --- | --- | --- | --- | --- |
| PC1 | `43.109.23.12/8` | A | `43.0.0.0` | `43.255.255.255` | `43.0.0.1` | `43.255.255.254` | `16,777,214` |
| PC4 | `129.221.23.13/16` | B | `129.221.0.0` | `129.221.255.255` | `129.221.0.1` | `129.221.255.254` | `65,534` |
| PC8 | `209.211.3.22/24` | C | `209.211.3.0` | `209.211.3.255` | `209.211.3.1` | `209.211.3.254` | `254` |
| PC5 | `2.71.209.233/8` | A | `2.0.0.0` | `2.255.255.255` | `2.0.0.1` | `2.255.255.254` | `16,777,214` |
| PC6 | `155.200.201.141/16` | B | `155.200.0.0` | `155.200.255.255` | `155.200.0.1` | `155.200.255.254` | `65,534` |

---

## ⚙️ Configuring IP Addresses on Cisco Devices

### 1. 🛠️ Basic Interface Configuration

Configure IPs in:

**Global Configuration mode → Interface Configuration mode**

```
Router> enable
Router# configure terminal
Router(config)# interface gigabitethernet 0/0
Router(config-if)# ip address 10.0.0.1 255.255.255.0
Router(config-if)# no shutdown
```

✅ You can use either:

- **Dotted decimal:** `255.255.255.0`
- **Prefix length:** `/24` where supported

Example with prefix notation:

```
Router(config-if)# ip address 10.0.0.1/24
```

> ⚠️ `no shutdown` is critical — router interfaces are **administratively down by default**.
> 

---

### 2. 🏷️ Adding Interface Descriptions

Good practice for documentation — helps you remember what the interface connects to.

```
Router(config-if)# description ## Link to Switch1 ##
```

---

### 3. 🔍 Verification Commands

```
Router# show ip interface brief
Router# show interfaces
Router# show interfaces description
```

| Command | What It Shows |
| --- | --- |
| `show ip interface brief` | Quick summary of all interfaces, IP addresses, and up/down status |
| `show interfaces` | Detailed Layer 1 / Layer 2 stats: speed, duplex, MAC, errors, bandwidth |
| `show interfaces g0/0` | Detailed stats for a specific interface |
| `show interfaces description` | Interface descriptions and status |

---

## 🧰 Extra Useful Syntaxes

```
! View running configuration for one interface
Router# show running-config interface gigabitethernet 0/0

! View detailed IP-related interface info
Router# show ip interface gigabitethernet 0/0

! From config mode, run show commands with do
Router(config)# do show ip interface brief

! Save configuration
Router# write memory
! or
Router# copy running-config startup-config
```

---

## 🎥 Video Reference

- **Review of Classes / Hosts:** `1:19–3:03`
- **Usable Address Calculations:** `5:39`
- **`show ip interface brief`:** `9:34`
- **Assigning IP Addresses:** `13:20`
- **`show interfaces`:** `20:36`
- **Interface Descriptions:** `22:54`
- **Quiz:** `25:04–28:50`

---

## ✅ Quick Recap

- Use the **Big 5** to find network, broadcast, first/last usable, and max hosts.
- Host bits determine everything: `0` = network, `1` = broadcast.
- Max hosts = `2^h − 2`.
- Configure router IPs under the interface with `ip address`, then `no shutdown`.
- Use `description` for documentation.
- Verify with `show ip interface brief`, `show interfaces`, and `show interfaces description`.