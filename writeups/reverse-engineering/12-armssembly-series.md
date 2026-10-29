# CyLab Academy — ARMssembly 2/3/4 · Writeup Guide

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2021) |
| Tags | aarch64, ARM assembly, calling convention |
| Status | Solved |

> **Goal:** one guide for the ARMssembly series (2, 3, 4 — after 1's intro): read ARM64 assembly, compute the function of its arguments, submit a decimal number. Different architecture, same discipline as the asm series — plus the ARM calling convention that makes it readable.

---

## 1. Understanding the challenge
Each level ships an ARMv8/aarch64 `.S` file defining a function over one or more integer arguments (the arguments are given in the challenge text). Submit the decimal result.

## 2. Tools
- ARM64 ISA reference (the common-op table below covers ~90% of it)
- `gcc`/`qemu` (optional execution check via cross-compilation)
- Pen and paper for the register trace

## 3. Recon — the ARM64 calling convention
```text
arg1 -> w0/x0      arg2 -> w1/x1      arg3 -> w2/x2
return value -> w0/x0
w0 = low 32 bits of x0 (unsigned ops use w regs, signed context decides extension)
```
`stp x29, x30, [sp, -16]!` / `ldp ...` prologue/epilogue = stack frame save/restore. Register the args, then read top to bottom.

**[TODO]** screenshot: the provided ARMssembly snippet.

## 4. Solution Path
**Core op table you need:**
```text
mov w?, #imm        ldr w?, [x?, ...]        str
add w?, w?, w?      sub w?, w?, w?           mul w?, w?, w?
sdiv/udiv           cmp w?, w?               cset w?, lt/gt/eq   (branchless select)
b.cond label        b label                  bl func             ret
lsl/lsr w?, w?, #n  and/orr/eor w?, w?, w?
```
1. Trace registers line by line, substituting the given arguments.
2. Series patterns to recognize:
   - **ARSsembly 2**: an additive/multiplicative loop (`sub` + `cbnz`/`b.gt`) — compute iteration count, then the closed form.
   - **ARSsembly 3**: comparison/selection — `cmp` + `cset` picks the max/min of the two arguments (often with an offset). The answer is just an expression in the inputs.
   - **ARSsembly 4**: memory: values are loaded from a data section or computed through `lsl`/`mul` chains — track both the registers and any `[x?]` loads.
3. Mind the width: if the challenge passes values that overflow 32 bits in your mental math, the `w`-register (32-bit) result wraps — compute modulo 2^32 when needed.
4. Submit as a plain decimal integer (negative results allowed — watch `subs` vs `sub`, `sdiv` vs `udiv`).

Verification (optional but cheap on a Mac/Linux box with aarch64, or via qemu-user):
```bash
aarch64-linux-gnu-gcc harness.c armssemblyN.S -o run
qemu-aarch64 ./run
```

**[TODO]** evidence: register trace + final decimal for each level.

## 5. Commands / Payloads Used
```text
w0/w1 = args, w0 = return       # the convention that starts every read
cmp + cset                      # ARM's branchless if — read as ternary
```

## 6. Flag
```
**[TODO]** decimal answers for ARMssembly 2, 3, 4
```

## 7. Write It Up (report-ready paragraph)
> "Analysis of aarch64 routines required applying the ARM64 calling convention and control-flow semantics to reconstruct the computed function (<evidence>). The exercise maps to real-world mobile binary review: Android native libraries on arm64 devices present exactly this instruction set, and reading register-level logic is the baseline skill for validating vendor library behavior during security assessments."

## 8. Lessons
- ARM64 reads *cleaner* than x86-32: fixed 4-byte instructions, args in registers, `cmp`+`cset` instead of conditional-jump spaghetti.
- `cset w?, cond` is ARM's ternary — whenever you see it, the whole level is usually "compute max/min/abs of the inputs".
- Keep one personal page: the op table above + the convention. It decodes the whole series and most arm64 native snippets you will ever grep in Android work.
