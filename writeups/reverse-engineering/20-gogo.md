# CyLab Academy — gogo · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2022) |
| Tags | golang, ghidra, symbol recovery, goroutines |
| Status | Solved |

> **Goal:** a stripped-ish Go binary checks your input through several goroutines. Go binaries are their own dialect — recover the runtime symbols, find `main.*`, and read the comparison logic.

---

## 1. Understanding the challenge
A Go-compiled ELF. Go retains far more metadata than C binaries: function names, type strings, and error messages survive even in "stripped" builds because the runtime needs them. That metadata is your map. The logic itself typically spreads across goroutines that each transform or check part of your input.

## 2. Tools
- Ghidra (with Golang analyzer; recent Ghidra versions handle Go well)
- `strings` / `grep 'main\.'`
- `go tool objdump` or `objdump -d` (Ghidra alternative for quick looks)
- gdb (GEF/pwndbg) for runtime confirmation

## 3. Recon
```bash
file gogo                       # ELF x86-64, "Go BuildID=..." usually visible
strings gogo | grep -i 'main\.' # recovers main.main, main.checkXXX, ... even when stripped
strings gogo | grep -iE 'flag|correct|wrong|pico'
go version -m gogo              # prints the Go toolchain version that built it
```
Load into Ghidra: let the Golang analyzer recover function symbols, then filter the symbol tree for `main`. Your target set is small — a `main.main` and a handful of `main.<helper>` functions.

**[TODO]** screenshot: the recovered `main.*` function list in Ghidra.

## 4. Solution Path
1. **Read `main.main`** — find where your input is taken and which helper functions receive slices of it. Go calling convention (recent toolchains): arguments and returns on the stack (`0x8(SP)`, `0x10(SP)`...), which makes the decompiles noisy but traceable.
2. **Read each helper** — the typical shape: each goroutine/helper compares a slice or transformed chunk of your input against an embedded constant (byte arrays, XOR'd values, or a shuffled order). Collect every expected chunk with its transform:
   - Byte arrays in the decompile are direct — transcribe them.
   - Transforms (rotation, XOR, reversal) are a handful of Go idioms: `+=` loops, `[]byte(input)[i]`, `time.Sleep` only as scheduling noise — ignore sleeps.
3. **Reassemble** the chunks in the order the main function concatenates/compares them, applying inverse transforms as needed:
```python
parts = [bytes([0x70, 0x69, ...]), ...]   # transcribed from each helper
print(b''.join(parts))                    # or apply the seen transform inversely first
```
4. **Confirm at runtime** — feed the candidate to the binary; the success string prints.
```bash
./gogo '<candidate>'
```

Why it works: Go's runtime keeps enough symbols and type info to make reversing approachable, and goroutine "parallelism" is cosmetic — the checks are sequential logic in disguise, each one localized and small.

**[TODO]** evidence: the helper comparisons + the success run.

## 5. Commands / Payloads Used
```bash
strings gogo | grep 'main\.'
go version -m gogo
./gogo '<reassembled-input>'
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "A Go binary validated user input across multiple helper routines (<evidence>). Runtime-retained symbol metadata enabled recovery of the application-level functions, and static reading of the embedded comparison constants allowed reconstruction of the accepted input. Go binaries expose substantially more structure than equivalent stripped C builds, reducing attacker effort; compiled client-side validation of secrets remains recoverable regardless of language. Remediation: treat any embedded credential/key as public and validate server-side."

## 8. Lessons
- `strings | grep 'main\.'` is the 10-second triage for any Go challenge — it lists the whole attack surface before Ghidra even opens.
- Ignore the noise: goroutines, `time.Sleep`, scheduler calls — the logic is in `main.*` byte comparisons.
- Go stack-based calling convention makes decompiles uglier than C; when stuck, cross-check registers/stack slots in the raw disassembly.
