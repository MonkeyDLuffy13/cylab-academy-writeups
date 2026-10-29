# CyLab Academy — Easy as GDB · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium (picoCTF 2021) |
| Tags | gdb python api, brute force, character oracle, breakpoints |
| Status | Solved |

> **Goal:** the flag "has got to be checked somewhere" — and it is checked **one character at a time**. A GDB Python script that watches the comparison loop turns the check into a character oracle and extracts the flag letter by letter.

---

## 1. Understanding the challenge
A binary `brute` asks for the flag and answers Correct/Incorrect. Ghidra shows the shape:
- `main` reads your input, applies an encoding function, calls `check_flag(user_input, len)`.
- `check_flag` re-decodes your input and compares it byte-by-byte against a global `g_encoded_flag` blob:

```c
while (true) {
    if (encoded_flag_len <= i) return 1;
    if (__dest[i] != __dest_00[i]) break;   // <-- per-character comparison
    i++;
}
return 0xffffffff;
```

The moment a character is wrong, the loop breaks. That early-exit is the oracle: **the loop runs longer when your prefix is correct.**

## 2. Tools
- gdb with GEF/pwndbg + the **GDB Python API** (`gdb.Breakpoint` subclasses)
- Ghidra (to locate the comparison instruction)

## 3. Recon
1. Run once: `input the flag: test` -> `checking solution... Incorrect.`
2. Ghidra: find the comparison assembly inside `check_flag`:
```asm
MOVZX EDX, byte ptr [EAX]        ; your encoded char
MOVZX EAX, byte ptr [ECX]        ; expected char
CMP  DL, AL                      ; <-- breakpoint here
JZ   next_iteration
```
3. Note the address of the `CMP` (e.g. `0x5655598e` after disabling ASLR / consistent PIE base) and the address of the `"Correct!"` push.

**[TODO]** screenshot: the comparison loop in Ghidra with the CMP address marked.

## 4. Solution Path
1. **Script the oracle.** Two breakpoint classes:
   - `Checkpoint` at the `CMP`: on hit number `len(flag)+1` (skip hits consumed by already-known characters), compare `$al == $dl` and report through a queue.
   - `Solvepoint` at the `Correct!` push: counts terminal hits to know when to stop.
2. **Brute force alphabet by alphabet**, one position at a time:
```python
import gdb, string
from queue import Queue

ALPHABET = string.ascii_letters + string.digits + "{}_"

class Checkpoint(gdb.Breakpoint):
    def __init__(self, queue, target_hitcount, *args):
        super().__init__(*args)
        self.silent = True; self.queue = queue; self.target = target_hitcount; self.hit = 0
    def stop(self):
        self.hit += 1
        if self.hit == self.target:
            self.queue.put(gdb.parse_and_eval("$al") == gdb.parse_and_eval("$dl"))
        return False

flag = ""
gdb.execute("set disable-randomization on")
queue = Queue()
for i in range(len(flag), 64):
    for c in ALPHABET:
        bp = Checkpoint(queue, len(flag) + 1, "*0x5655598e")
        gdb.execute("run <<< {}{}".format(flag, c))
        if queue.get(timeout=1):
            flag += c
            print(flag)
            break
        bp.delete()
gdb.execute("q")
```
3. Each iteration appends one character; after ~30 iterations the whole flag is on screen.

Why it works: per-character comparison with early exit leaks how much of your guess is right — the same side channel as timing attacks on `strcmp`. Encoding does not matter; the oracle sits after it.

**[TODO]** evidence: the flag growing character by character in the script output.

## 5. Commands / Payloads Used
```python
gdb.Breakpoint subclass + queue.get()          # per-hit conditional logic
gdb.execute("run <<< <prefix><candidate>")
$al == $dl                                     # the character oracle
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The binary validated input through a byte-wise comparison loop with early termination (<evidence>). A GDB Python script used hit-counted breakpoints at the comparison instruction to build a character oracle and recover the flag one character at a time. Byte-at-a-time validation is a classic weakness independent of the encoding applied beforehand. Remediation: compare constant-time over the whole value (or move validation server-side), never short-circuit on the first differing character in security-relevant checks."

## 8. Lessons
- `gdb.Breakpoint` subclasses + `queue` turn gdb into a programmable oracle — a template reusable for OTP Implementation (writeup 26) and any char-by-char check.
- Hit-counting matters: the breakpoint fires once per known character; only the next hit carries new information.
- The challenge name is the hint: when something is "easy as GDB", don't reverse the encoder — automate the debugger.
