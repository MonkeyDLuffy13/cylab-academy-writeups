# CyLab Academy — Need For Speed · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | gdb, binary patching, anti-unko (time gate) |
| Status | Solved |

> **Goal:** the gate is *time*, not data: the binary insists on a slow drive before decrypting the flag. Defeat it by controlling execution in gdb (or patching the sleep out) — the standard answer to every "wait N seconds/days" challenge.

---

## 1. Understanding the challenge
`./need-for-speed` prints an accelerating ASCII car and stalls on sleeps between "gears", finally hinting that the flag decrypts only after the run completes. In the binary, the flow is: sleep-loop -> `decrypt_flag(key, len, ...)` -> print. Nothing about your input matters; only patience or interference does.

## 2. Tools
- gdb
- `strings` / disassembly for recon
- Optional: patchelf / hex editor for the patch route

## 3. Recon
```bash
strings need-for-speed | grep -iE 'flag|speed|gear|accel'
objdump -d need-for-speed | grep -A20 -iE 'decrypt|main'
```
Locate: the sleep call(s) (`usleep`/`sleep`/`nanosleep`) and the call to `decrypt_flag`. Note its address and the argument setup (the key register).

**[TODO]** screenshot: disassembly around decrypt_flag.

## 4. Solution Path
**Route A — gdb, no patching:**
```bash
gdb ./need-for-speed
(gdb) break decrypt_flag        # symbol or address
(gdb) run
(gdb) info args                 # see the key it is about to use
(gdb) continue                  # the decrypt prints the flag
```
The sleep loop is skipped because the breakpoint lands *after* it. If `decrypt_flag` is static/inlined, break on the `puts`/`printf` that follows the loop instead.

**Route B — patch the sleeps out:**
1. Find the sleep call sites (`objdump -d | grep -B2 'sleep'`).
2. NOP the call (x86: write `0x90`s over the `call` bytes) with a hex editor, save, run — the "drive" finishes instantly.

**Route C — let it actually drive:** some builds finish in a tolerable couple of minutes; just wait and watch. (Try this first — it is the zero-effort sanity check.)

Why it works: time-gating is client-side policy. Any process you control can be stopped at, stepped through, or edited — "wait 2 minutes" is only a barrier for environments you do not control.

**[TODO]** evidence: gdb session hitting decrypt_flag + the flag output.

## 5. Commands / Payloads Used
```bash
gdb -ex 'break decrypt_flag' -ex run ./need-for-speed
0x90 0x90 0x90 ...             # NOP sled over the sleep call (patch route)
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The application gates functionality behind elapsed-time conditions enforced in process (<evidence>). Debugger control-flow manipulation (or binary patching of the delay calls) bypassed the gate entirely, reaching the protected decryption routine. Time- and resource-based client checks are availability signals only; durable controls must be enforced where the attacker has no execution control (server-side)."

## 8. Lessons
- Breakpoint after the gate beats fighting the gate. Find where the interesting function *starts* and set up camp there.
- NOPing a call (`0x90` fill) is the minimal-edit patch; keep a backup copy of any binary you patch.
- This pattern (sleep/check gates) returns constantly in licensing, trialware, and game RE — same three routes every time.
