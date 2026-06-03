# Compiled — TryHackMe

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
- A compiled binary was provided requiring the correct password as input
- No network services involved — purely static binary analysis
- Approach: reverse engineering with Ghidra to inspect the decompiled logic

---

## 📡 Enumeration

> Goal: extract as much information as possible from the binary.

### Static Analysis with Ghidra

The binary was loaded into Ghidra for decompilation. Inside the `main` function, the following logic was identified:

```c
fwrite("Password: ", 1, 10, stdout);
__isoc99_scanf("DoYouEven%sCTF", local_28);

iVar1 = strcmp(local_28, "__dso_handle");
if ((-1 < iVar1) && (iVar1 = strcmp(local_28, "__dso_handle"), iVar1 < 1)) {
  printf("Try again!");
  return 0;
}

iVar1 = strcmp(local_28, "_init");
if (iVar1 == 0) {
  printf("Correct!");
} else {
  printf("Try again!");
}
```

**Key observations:**
- `scanf` reads input using the format `DoYouEven%sCTF`, capturing the middle portion into `local_28`
- First `strcmp` rejects `__dso_handle`
- Final `strcmp` accepts `_init` as the correct value

---

## 💥 Exploitation

> Goal: determine and supply the correct password.

**Vulnerability found:** Hardcoded password comparison in decompiled binary logic

**Tools used:** Ghidra, basic C analysis

### Code Interpretation

The `scanf` format string `DoYouEven%sCTF` means the program expects full input in the format:

```
DoYouEven<VALUE>CTF
```

The `%s` portion is extracted into `local_28` and compared via `strcmp`. The misleading first comparison against `__dso_handle` is a red herring — the actual acceptance condition is:

```c
strcmp(local_28, "_init") == 0
```

### Solution

The correct password is:

```
DoYouEven_initCTF
```

Running the binary:

```bash
./compiled
Password: DoYouEven_initCTF
Correct!
```

---

## 🚩 Flags

| Flag | How Found |
|------|-----------|
| `Correct!` output | Supplied password `DoYouEven_initCTF` derived from Ghidra decompilation |

---

## 📚 Lessons Learned

1. **The final condition is what matters** — misleading comparisons earlier in the code can distract from the actual acceptance logic
2. **scanf format strings reveal input structure** — `DoYouEven%sCTF` directly tells you how to construct the full input
3. **Ghidra decompiles binaries into readable C** — even without source code, the logic is recoverable through static analysis
4. **Red herrings are common in RE challenges** — always trace all branches before concluding what the correct input is

---

## 🛠️ Tools Used

- `Ghidra` (static binary analysis / decompilation)
- Basic C code analysis

---

*Writeup by GabrielMachado0*
