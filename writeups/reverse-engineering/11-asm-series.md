# CyLab Academy — asm 2/3/4 · Writeup Guide

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | x86 assembly, cdecl, static analysis |
| Status | Solved |

> **Goal:** one guide for the whole `asmN` series: you get a fragment of x86 (32-bit) assembly computing a function of its arguments, and must submit the numeric result. The skill is reading calling conventions and walking registers — plus a verification trick that turns the challenge into a compile-and-run exercise.

---

## 1. Understanding the challenge
Each `asmN` provides a `.S`/text assembly snippet defining a function (`asm2(0x?,0x?)`, `asm3(0x?,0x?,0x?)`, `asm4("<string>")`). The answer is the return value in decimal (submitted as a number).

## 2. Tools
- Any x86 ISA reference (or an LLM-free cheat sheet in your notes)
- `gcc -m32` + a C harness (verification)
- gdb (optional single-step)

## 3. Recon — the calling convention that unlocks everything
32-bit cdecl: arguments live on the stack. After the standard prologue (`push ebp; mov ebp, esp`):
```text
arg1 = [ebp+8]     arg2 = [ebp+12]     arg3 = [ebp+16]
return value -> eax
```
Identify per challenge: which args are read, whether they are treated as 32-bit words, 16-bit halves (`ax`), or bytes (`al/bl/cl/dl` — asm3's signature trick), and whether memory/string args are dereferenced (asm4).

**[TODO]** screenshot: the provided asmN snippet.

## 4. Solution Path
**Method 1 — walk it by hand** (the point of the exercise):
1. Annotate each instruction with the register/stack state after it.
2. Loops (`dec jnz` / `add loop`): compute iterations, then the closed form (asm2 is typically an additive loop over one argument in steps of the other).
3. Byte-level accesses: remember `[reg]` addressing and that `al`/`bl` touch only the low byte — sign/zero extension (`movzx` vs `movsx`) decides whether values go negative.
4. asm4 calls a helper over a string: trace the helper first (usually a per-character accumulator), then run the outer arithmetic.

**Method 2 — compile and run it** (verification, and a life skill):
```c
// harness.c
#include <stdio.h>
int asm2(int, int);            // matches the provided symbol
int main() { printf("%d\n", asm2(0xb, 0x2e)); return 0; }
```
```bash
gcc -m32 harness.c asm2.S -o run && ./run       # prints the answer in decimal
```
This works because the challenge ships *real* assembly — assembling it against a harness executes the truth instead of your reading of it. (Keep the by-hand answer too; comparing the two is where the learning happens.)

**[TODO]** evidence: your hand-walk notes + the harness output for each of asm2/3/4.

## 5. Commands / Payloads Used
```bash
gcc -m32 harness.c asmN.S -o run && ./run
gdb ./run --ex 'break asmN' --ex 'run' --ex 'info registers'
```

## 6. Flag
```
**[TODO]** the decimal answers for asm2, asm3, asm4 (submit each as plain numbers)
```

## 7. Write It Up (report-ready paragraph)
> "Reverse engineering of the provided x86 routines required reconstructing behavior from register/stack semantics under cdecl (<evidence>). Execution-based verification (assembling the supplied code against a C harness) confirmed manually derived results, illustrating the dual static/dynamic workflow used in binary analysis: hypothesize statically, verify dynamically. The same loop applies when auditing vendor-supplied binaries during incident response."

## 8. Lessons
- cdecl offsets `[ebp+8/12/16]` are worth burning into memory — they decode every asmN instantly.
- Watch data width: word (`eax`) vs byte (`al`) reads are where asm3-style challenges hide their twist.
- When allowed to execute, never hand-verify what you can compile — but do both once, for the learning.
