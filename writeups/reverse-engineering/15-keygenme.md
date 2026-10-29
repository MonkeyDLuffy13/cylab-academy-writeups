# CyLab Academy — Keygenme · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2022) |
| Tags | ghidra, keygen, license check |
| Status | Solved |

> **Goal:** a classic license-check crackme: the binary validates a key you must *generate*, not find. Decompile with Ghidra, extract the per-character validation, and write a keygen — the difference between this and vault-door is that the transformation inverts into a generator.

---

## 1. Understanding the challenge
`keygenme` (a binary, not source) prompts for a key. The expected key follows the flag format `picoCTF{...}` internally, and the check validates it piecewise (format prefix, then transformed characters). Your deliverable is a valid key — and ideally a script that produces them.

## 2. Tools
- Ghidra (decompiler)
- `strings` (quick constants sweep)
- Python (keygen)

## 3. Recon
```bash
strings keygenme | grep -iE 'pico|key|valid|invalid'
```
Load in Ghidra -> locate `main` -> follow to the check function. Typical shape after decompilation:
```c
if (input[0] != 'p' || input[1] != 'i' || ...) return 0;   // literal prefix
// then: transformed = (input[i] - base) ^ k  compared against table[i]
// or:    sum of transformed chars must equal constant
```
**[TODO]** screenshot: the decompiled check function.

## 4. Solution Path
1. Classify the check from the decompilation:
   - **Literal prefix/suffix** -> copy as-is.
   - **Per-character arithmetic** (`xor/add/sub/rot` against constants or a table) -> invert per character.
   - **Aggregate condition** (sum/hash equals C) -> constrain and search (small spaces: brute force; larger: z3).
2. Mirror the check in Python, inverted:
```python
table = [0x??, ...]                       # from the decompilation
key_body = ""
for i, t in enumerate(table):
    key_body += chr(((t ^ 0x??) - 0x??) % 256)     # invert the ops in reverse order
print("picoCTF{" + key_body + "}")
```
3. If an aggregate condition binds the whole key, hand the constraints to z3:
```python
from z3 import *
s = Solver(); key = [BitVec(f'k{i}', 8) for i in range(N)]
for c in key: s.add(c >= 32, c < 127)
s.add(Sum(key) == 0x??)                    # transcribe the real condition
s.check(); print(bytes([s.model()[c].as_long() for c in key]))
```
4. Verify: run the binary with the generated key -> success branch -> flag.

Why it works: any check that admits a satisfying key admits a *generator* for such keys. Crackme-vs-keygen is only the direction you walk through the same math.

**[TODO]** evidence: decompiled check + keygen output accepted by the binary.

## 5. Commands / Payloads Used
```python
inverse-ops per char        # xor/add chains: undo in reverse order
z3 Sum/key constraints      # for aggregate conditions
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The distributed binary enforces license validation through locally checkable key properties (<evidence>). Decompilation exposed the validation logic, and its inversion (or constraint solving) produced a working key generator, bypassing the licensing control entirely. Remediation: license checks on user-controlled machines are advisory — bind entitlements to server-issued, signed tokens and verify them server-side for anything of value."

## 8. Lessons
- Ghidra default analysis finds `main` in stripped CTF binaries most of the time; from there, follow the branch on the verdict string.
- Decision tree for key checks: per-char invertible -> mirror inverse; aggregate -> z3; tiny space -> brute force.
- Keep a personal `z3_scaffold.py` (BitVecs, bounds, Sum) — it returns in crackmes, crypto constraints, and even SAT-style forensics puzzles.
