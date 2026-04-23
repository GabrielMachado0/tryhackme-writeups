# Compiled – TryHackMe

## Objective
Analyze a compiled binary to determine the correct password using reverse engineering techniques.

---

## Initial Approach

The challenge required identifying the correct input accepted by a compiled program.

The initial approach involved analyzing the binary using **Ghidra** to understand how input validation was implemented.

---

## Methodology

### Step 1 — Static Analysis with Ghidra

The binary was loaded into Ghidra to inspect the decompiled code and identify the program logic.

---

### Step 2 — Identifying the Main Function

Inside the `main` function, the following code was observed:

```c
fwrite("Password: ",1,10,stdout);
__isoc99_scanf("DoYouEven%sCTF",local_28);

iVar1 = strcmp(local_28,"__dso_handle");
if ((-1 < iVar1) && (iVar1 = strcmp(local_28,"__dso_handle"), iVar1 < 1)) {
  printf("Try again!");
  return 0;
}

iVar1 = strcmp(local_28,"_init");
if (iVar1 == 0) {
  printf("Correct!");
}
else {
  printf("Try again!");
}
```

### Step 3 — Code Interpretation

The program reads user input using scanf
The expected format appears to be:
```c
DoYouEven<INPUT>CTF
```
The input is then compared using strcmp:

First comparison rejects "__dso_handle".

Final comparison checks if the input equals "_init"
Solution.

The correct password accepted by the program is:
```bash
DoYouEven_init
```
### Key Concepts Learned
Understanding how strcmp works in C
Interpreting conditional logic in compiled programs.
Using Ghidra for basic reverse engineering
Recognizing misleading patterns in decompiled code.

### What I Learned
This challenge highlighted that:
The most important logic is often found in the final condition, not necessarily in how input is read.
Even if the scanf format suggests a specific structure, the actual validation logic determines the correct input.

### Tools Used
```bash
Ghidra
Basic C code analysis
```
