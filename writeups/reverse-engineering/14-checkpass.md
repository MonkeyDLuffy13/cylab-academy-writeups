# CyLab Academy — Checkpass · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2021) |
| Tags | angr, symbolic execution, constraint solving |
| Status | Solved |

> **Goal:** the check function is deliberately convoluted (byte-wise arithmetic soup) — inverting it by hand is the trap. The intended (and industry-real) tool: **angr**, which explores execution symbolically and solves for the input that reaches the "Correct" branch.

---

## 1. Understanding the challenge
A binary reads a password and validates it through a tangle of arithmetic and comparison operations, then prints success/failure. The transformation is bijective but a nightmare algebraically — and trivial for a solver.

## 2. Tools
- angr (`pip install angr`)
- The success/failure strings from `strings` (addresses needed by the script)

## 3. Recon
```bash
strings checkpass | grep -iE 'correct|wrong|nice|nope'
objdump -d checkpass | grep -B5 -A5 'correct'      # or load in Ghidra quickly
```
Note two addresses:
- the address of the block that prints success ("Good job" / flag path),
- the address that prints failure.
Also note how input enters (argv or stdin — changes the script's entry factory).

**[TODO]** screenshot: the success/failure strings and their addresses.

## 4. Solution Path
The standard angr template (adapt the three marked lines):
```python
import angr, claripy

proj = angr.Project('./checkpass', auto_load_libs=False)

argv1 = claripy.BVS("argv1", 8*16)                 # 16-byte symbolic password
state = proj.factory.entry_state(args=['./checkpass', argv1])
                     # ^ if input is stdin instead, build a filestate:
                     #   simgr = proj.factory.simgr(state) with a file stream

simgr = proj.factory.simulation_manager(state)
good = 0x<success-addr>
bad  = 0x<failure-addr>
simgr.explore(find=good, avoid=bad)

if simgr.found:
    s = simgr.found[0]
    print(s.solver.eval(argv1, cast_to=bytes))     # the solved password
```
```bash
python3 solve.py
./checkpass '<solved-password>'     # verify against the real binary
```
Why it works: angr treats your input as symbolic variables and the program as constraints; `explore` steers execution away from the failure address and toward success, then the SMT solver produces concrete bytes satisfying every check on the path. Whatever arithmetic sits between input and verdict becomes one big constraint expression.

**[TODO]** evidence: solver output + the binary accepting the password.

## 5. Commands / Payloads Used
```python
claripy.BVS / simgr.explore(find=..., avoid=...) / solver.eval(...)
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The binary validates credentials through an obfuscated arithmetic chain equivalent to a system of constraints (<evidence>). Symbolic execution with an SMT backend recovered a satisfying input in seconds, demonstrating that key-strength for validation logic is algorithmic transparency, not operation count. Remediation: enforce authentication where solver-grade analysis does not apply (server-side, rate-limited, salted hashes); client checks of any complexity yield to symbolic execution."

## 8. Lessons
- angr's shape never changes: project -> symbolic input -> explore(find/avoid) -> eval. Save the template; it solves half of all "crackme" challenges.
- `find`/`avoid` addresses are the only per-challenge work — `strings` + Ghidra gets them in two minutes.
- When hand-inverting a transform feels long, ask: is this per-char independent (brute force, vault-door-8) or path-dependent (symbolic execution, this)? Picking the right solver class IS the solve.
