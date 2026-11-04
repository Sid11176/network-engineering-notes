# Introduction to CLI (10.23.2026)

**Cisco IOS CLI & Privileged EXEC Security — Class Notes**

---

## 1. Cisco IOS Command Mode Flow

```
Router>                          User EXEC mode
  │ enable
  ▼
Router#                          Privileged EXEC mode
  │ configure terminal
  ▼
Router(config)#                  Global configuration mode
```

**Mode prompts at a glance:**

| Prompt | Mode |
| :--- | :--- |
| `Router>` | User EXEC mode |
| `Router#` | Privileged EXEC mode |
| `Router(config)#` | Global configuration mode |

> ⚠️ **Important:** `enable password` and `enable secret` are **not** global configuration commands, so they must be entered at `Router(config)#`, **not** directly at `Router#`.

---

## 2. Enable Password vs. Enable Secret

Both commands set the password requested when a user runs `enable` to enter privileged EXEC mode. But they differ in security.

| Feature | `enable password` | `enable secret` |
| :--- | :--- | :--- |
| **Prompted at** | `enable` | `enable` |
| **Default storage** | Clear text | Hashed |
| **With `service password-encryption`** | Type 7, weak/reversible | Unaffected |
| **Security level** | Weak | Strong |
| **If both configured** | Ignored | Takes precedence |
| **Recommended?** | Avoid | Use this |

> **Key point:** If both are configured, **only the `enable secret` password** is accepted when entering privileged EXEC mode.

---

## 3. Corrected Command Reference

| Command | Correct mode | Purpose |
| :--- | :--- | :--- |
| `enable` | User EXEC `>` | Enter privileged EXEC mode; prompts for enable secret/password |
| `configure terminal` | Privileged EXEC `#` | Enter global configuration mode |
| `enable password PASSWORD` | Global config `(config)#` | Set weak enable password |
| `enable secret PASSWORD` | Global config `(config)#` | Set strong hashed enable secret |
| `service password-encryption` | Global config `(config)#` | Encrypts clear-text passwords using weak type 7; does **not** protect `enable secret` |
| `no <command>` | Global config `(config)#` | Removes a configured command |
| `show running-config` | Privileged EXEC `#` | Display active configuration |
| `show startup-config` | Privileged EXEC `#` | Display saved configuration |
| `write memory` | Privileged EXEC `#` | Save configuration to NVRAM |
| `copy running-config startup-config` | Privileged EXEC `#` | Save configuration; preferred modern command |

### Important Correction to Your Notes

Your note had:

```
Router(config)# run privileged-exec-level-command
```

❌ That is **not** a real Cisco IOS command.

From global configuration mode, you **can** run privileged EXEC commands using `do`:

```
Router(config)# do show running-config
Router(config)# do write memory
```

So replace that line with:

```
Router(config)# do show running-config
## runs a privileged EXEC command from global configuration mode
```

> `show`, `write`, and `copy` are privileged EXEC commands. From `(config)#`, use `do ...` or `exit` back to `#`.

---

## 4. Console Connection Cable

```
RJ45 console port  →  rollover cable
```

- A Cisco RJ45 console port uses a **rollover cable**.
- Often light blue, RJ45-to-RJ45, with a DB9/USB adapter.
- **Not** a crossover cable.
- Modern devices may also offer mini-USB/USB console ports.

---

## 5. Quiz Answer Key

**1. What kind of cable is used to connect to a Cisco device via the RJ45 console port?**
> **a) Rollover cable**

**2. You type `enable`, but the password is not accepted. What could be the problem?**
> **c) Caps Lock is on**
>
> *Real-world also check: wrong password, `enable secret` precedence, keyboard layout.*

**3. Most secure method to protect privileged EXEC mode?**
> **a) `enable secret`**

**4. If both `enable password` and `enable secret` are configured, what happens?**
> **c) You must enter the `enable secret` only.**

**5. Full version of `conf t`?**
> **b) `configure terminal`**

---

## 6. Extra Security Notes

- `service password-encryption` is **not strong security**. It uses reversible type 7 encryption.
- `enable secret` is automatically hashed when configured.
- Older IOS uses type 5 MD5; newer IOS supports stronger type 8/9 hashes. Where available, use stronger algorithms.
- You **cannot recover** an `enable secret` from the config; you can only reset/replace it.
- Prefer `enable secret` and avoid `enable password` entirely.

---

## 7. Quick Memory Summary

| Item | Summary |
| :--- | :--- |
| `enable password` | Weak, clear text / type 7 |
| `enable secret` | Strong, hashed, **wins if both exist** |
| `enable` prompt | Happens when you type `enable` |
| Both commands | Entered in global config mode (`(config)#`) |
| `show` / save / `copy` | Privileged EXEC; from config use `do ...` |
| Console cable | Rollover cable |