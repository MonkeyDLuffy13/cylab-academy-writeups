# CyLab Academy — riscy business · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium (picoMini by redpwn) |
| Tags | risc-v, assembly reading, function semantics |
| Status | Solved |

> **Goal:** a short RISC-V function — the challenge hands you assembly (or a `qemu` binary) and asks what it returns for a given argument. Same game as the ARMssembly series, different ISA: translate the instructions to arithmetic and evaluate.

---

## 1. Understanding the challenge
The RISC-V cousin of ARMssembly: a compact function (`func`-style) whose body is a dozen instructions. You either answer "what does it return for input X" or run it. RISC-V is the *easiest* mainstream ISA to read — small orthogonal instruction set, everything visible.

## 2. Tools
- A RISC-V instruction reference (the CS cheat-sheet class)
- `riscv64-linux-gnu-gcc` / `qemu-riscv64` (if a binary is provided)
- Python (for the final arithmetic)

## 3. Recon — the RISC-V reading rules
Everything you need for these challenges fits on one card:

| Concept | Rule |
|---|---|
| Args / return | `a0`-`a7` (x10-x17) carry args; `a0` returns |
| Register widths | `w`-suffix has no RISC-V equivalent: `a0` is 64-bit on RV64, 32-bit on RV32 (check the file/binary) |
| `li rd, imm` | load immediate |
| `mv rd, rs` | copy register |
| `add rd, rs1, rs2` / `addi rd, rs1, imm` | add (immediate variant) |
| `sub`, `mul`, `div` | arithmetic (signed); `divu/mulhu` unsigned |
| `slli/srli rd, rs, imm` | shift left/right logical by immediate |
| `srai` | arithmetic (sign-extending) right shift |
| `and/or/xor rd, rs1, rs2(/imm)` | bitwise |
| `beq rs1, rs2, label` (+`bne/blt/bge/bltu/bgeu`) | conditional branch |
| `jal rd, label` / `jalr` | call (rd = return address; `jal label` = `ra` implied) |
| `lw/sw` (+`ld/sd/lb/sb`) | load/store; width from the letter |
| Loop shape | label at the top + conditional branch back = loop |

**[TODO]** screenshot/paste: the provided RISC-V function.

## 4. Solution Path
1. **Number the instructions** and trace `a0`/temporaries through them like a spreadsheet — one column per register, one row per instruction, substituting the challenge's input value for `a0` at entry.
2. Watch the idioms:
   - `slli` by 1 = multiply by 2; `slli`+`add` pairs = multiply by 3 (leverage).
   - `srli` vs `srai`: sign matters only for negative values — with positive inputs they match.
   - `andi a0, a0, 0xff` style tail masks often indicate byte truncation.
   - `bltu/bgeu` compare *unsigned* — a negative-looking value becomes huge.
3. Compose the closed form and evaluate for the asked input:
```python
def func(a0):            # transcribed semantics
    return (a0 * 3) & 0xff   # example shape — replace with YOUR trace
print(func(<given input>))
```
4. If a binary was provided instead of a listing, verify:
```bash
qemu-riscv64 ./riscy <input>          # if it reads argv
# or compile your own reference from the transcript:
riscv64-linux-gnu-gcc -static func.S harness.c -o t && qemu-riscv64 ./t
```
   (Harness pattern identical to the asm-series writeups: `printf("%d\n", func(N));`)

Why it works: the function is pure arithmetic over its argument — a register trace with the immediate values plugged in *is* the answer, and the ISA card above covers 100% of the instructions these challenges use.

**[TODO]** evidence: your register trace + the final value.

## 5. Commands / Payloads Used
```bash
qemu-riscv64 ./riscy <input>
riscv64-linux-gnu-gcc -static func.S harness.c -o t && qemu-riscv64 ./t
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "A RISC-V function was provided for semantic analysis (<evidence>). The instruction sequence was traced register-by-register into an equivalent arithmetic expression, evaluated for the specified input, and cross-checked by native execution under QEMU user-mode emulation. Single-purpose assembly functions remain fully recoverable by inspection regardless of ISA; the methodology (trace registers, identify idioms, evaluate) transfers between x86, ARM, and RISC-V with only the convention table swapped."

## 8. Lessons
- RISC-V is the friendliest ISA to learn reading on: no flags register, no condition codes — every comparison is an explicit branch.
- Shifts-as-multiplication and unsigned-compare traps are the two "gotchas" planted in these challenges.
- Same skill spine as ARMssembly (writeup 12) — do them back-to-back and ISA stopped being scary.
