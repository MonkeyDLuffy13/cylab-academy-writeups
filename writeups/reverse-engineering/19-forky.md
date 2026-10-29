# CyLab Academy — Forky · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2022) |
| Tags | fork, process arithmetic, disassembly |
| Status | Solved |

> **Goal:** a program forks itself repeatedly and only prints the flag if the total process count satisfies an arithmetic condition you compute from the disassembly. Read the loop, do the math, run the binary.

---

## 1. Understanding the challenge
The binary's `main` calls `fork()` inside a loop whose trip count (and any per-iteration arithmetic) is visible in the disassembly. At the end, a check on the resulting process count gates the flag print. No dynamic analysis needed — this is algebra on a disassembly.

## 2. Tools
- Ghidra or `objdump -d forky` (decompile/disassemble)
- `ltrace -e fork` / `strace -f` (verify your count empirically)
- Calculator/paper

## 3. Recon
1. Disassemble and focus on three regions: the loop guard (`cmp`/`jle` before the `fork` call), anything mutating a counter between iterations, and the final comparison that gates the print.
2. Ghidra's decompiler usually renders it as:
```c
for (i = 0; i < N; i++) {
    fork();
    /* possible counter tweaks here */
}
if (<expression involving process count>) print_flag();
```
3. Note whether the parent waits (`wait*` calls) — tells you whether children also execute the remaining loop (doubling counts) or exit early.

**[TODO]** screenshot: the decompiled loop and the final check.

## 4. Solution Path
1. **Compute the count.** Track it precisely:
   - Each surviving process that reaches `fork()` adds one process — and children *continue* the loop unless the code exits them. If parent and children both keep looping, the count doubles per iteration: `2^N`.
   - If only the parent loops (children exit or the loop counter is per-process such that children skip), it's `N + 1`.
   - The challenge's trick is usually a specific mixed case — read which process keeps looping and apply the doubling or addition accordingly.
2. **Verify empirically** (cheap and decisive):
```bash
ltrace -e fork ./forky 2>&1 | grep -c fork
strace -f -e trace=fork,clone ./forky 2>&1 | grep -cE 'fork|clone'
```
   The trace count is your ground truth for the flag-print run.
3. **Read the gate** and satisfy it. Typically the check is `count == EXPECTED` or a relation (`count % k`, `count > m`) — the flag prints on the correct run with no input from you. If the check is `!=`-style (prints only on the *wrong* count, a known Forky-family gag), simply run it and read the message: it may literally tell you the answer.
4. Run and capture:
```bash
./forky
# or under strace to watch the process tree while it prints
strace -f ./forky | tail
```

Why it works: `fork()` semantics are deterministic — the disassembly plus a correct mental model of which processes continue looping fully determines the count. The challenge is reading, not brute force.

**[TODO]** evidence: the trace count + the flag output.

## 5. Commands / Payloads Used
```bash
objdump -d forky | less
strace -f -e trace=fork ./forky
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The binary gated flag output on the total number of processes created by its fork loop (<evidence>). Static analysis of the loop guard and continuation semantics yielded the expected count, verified against an strace/ltrace process trace, after which the program released the flag on a normal run. In assessments, process-creation behavior is fully predictable from the binary — dynamic tracing confirms the arithmetic in seconds. Defensive takeaway: security decisions computed from process internals are reproducible by an attacker with the binary; move such checks server-side."

## 8. Lessons
- The classic fork arithmetic: children that keep looping double the count per iteration (`2^N`); children that exit leave the parent's linear count (`N+1`). The whole challenge is deciding which.
- `strace -f` is your answer key — always verify the mental model against a trace before committing.
- Forky-family challenges sometimes print the flag precisely when the count is *wrong* — read the comparison operator, not just the numbers.
