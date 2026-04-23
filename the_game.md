# The Game – TryHackMe Lab

## Objective
Cipher has gone dark, but intel reveals he’s hiding critical secrets inside Tetris, a popular video game. Hack it and uncover the encrypted data buried in its code.

---

## Initial Approach

At first, the idea was to perform deeper reverse engineering using tools like Godot extraction.

However, the correct solution turned out to be much simpler: **static analysis using basic tools**.

---

## Methodology

### Step 1 — Isolation

The binary was analyzed inside a Kali Linux virtual machine to avoid running unknown executables on the host system.

---

### Step 2 — Initial Enumeration

The first approach was to inspect the binary using the `strings` command:

```bash
strings Tetrix.exe | grep THM
```

---

### Step 3 — Result

The flag was directly visible in the output:

```bash
THM{I_CAN_READ_IT_ALL}
```

---

### Key Findings
Sensitive data can be exposed in plain text inside binaries
Basic tools like strings can reveal critical information
Not all challenges require advanced reverse engineering

---

### Tools Used
```bash
Kali Linux
strings
grep
```

---

### What I Learned

This challenge reinforced an important principle in cybersecurity:
Always start with the simplest approach before moving to more complex tools.
Before using advanced tools like Ghidra or binary extractors, basic static analysis can often reveal useful information quickly.
