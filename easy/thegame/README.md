# The Game — TryHackMe

**Difficulty:** Easy  
**Category:** Reverse Engineering  
**Date:** 03/06/2026  
**Status:** ✅ Completed

---

## 📋 Table of Contents

- [Reconnaissance](#reconnaissance)
- [Enumeration](#enumeration)
- [Exploitation](#exploitation)
- [Flags](#flags)
- [Lessons Learned](#lessons-learned)

---

## 🔍 Reconnaissance

> Goal: understand the target binary and identify the analysis approach.

**Initial observations:**
- A Windows executable (`Tetrix.exe`) was provided — a Tetris-style game
- Intel suggests critical secrets are hidden inside the binary
- Analysis performed inside a Kali Linux VM to avoid running unknown executables on the host system

---

## 📡 Enumeration

> Goal: extract as much information as possible from the binary.

### Static Analysis with strings

Before reaching for advanced tools like Ghidra, basic static analysis was performed first:

```bash
strings Tetrix.exe | grep THM
```

**Result:** The flag was directly visible in plain text within the binary output.

---

## 💥 Exploitation

> Goal: extract the hidden flag from the binary.

**Vulnerability found:** Sensitive data stored in plain text inside the compiled binary

**Tools used:** `strings`, `grep`

No advanced reverse engineering was required. The flag was embedded as a readable string and immediately recoverable with basic enumeration.

```bash
strings Tetrix.exe | grep THM
THM{I_CAN_READ_IT_ALL}
```

---

## 🚩 Flags

| Flag | Location | How Found |
|------|----------|-----------|
| `THM{I_CAN_READ_IT_ALL}` | Plain text inside `Tetrix.exe` | `strings` + `grep` |

---

## 📚 Lessons Learned

1. **Always start simple** — `strings` and `grep` should be the first step before reaching for Ghidra or other heavy tools
2. **Sensitive data can be stored in plain text inside binaries** — developers sometimes hardcode strings that are fully recoverable without decompilation
3. **Isolation matters** — always analyse unknown executables inside a VM, never on the host system
4. **Not every RE challenge requires advanced techniques** — recognising when a simple approach is sufficient saves significant time

---

## 🛠️ Tools Used

- `strings`
- `grep`
- Kali Linux (isolated VM environment)

---

*Writeup by [GabrielMachado0](https://github.com/GabrielMachado0)*
