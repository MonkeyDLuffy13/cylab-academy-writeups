# CyLab Academy — OTP Implementation · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoMini 2020) |
| Tags | gdb python api, character oracle, xor, one-time pad |
| Status | Solved |

> **Goal:** a binary demands a 100-character hex key, "jumbles" it, and compares against a fixed target. The jumble is autoregressive (each output depends on the previous one) — which makes it a perfect character-by-character oracle for the same GDB brute-force template. Then XOR the recovered key with the provided hex blob.

---

## 1. Understanding the challenge
`./otp [KEY]` with a hex text file provided:
```
1fcb81cd1f6f1e12b429092e3647153b6c212772554ca004145b82367e1e6b7870827dc249a319601776f727434e6b6227d1
```
Ghidra on the binary shows three pieces:
1. `valid_char`: input alphabet is `[0-9a-f]` only — 16 candidates per position.
2. `jumble`: a bit-shifting transform of one character.
3. `main`: builds `jumbled_input[i]` from `jumble(user_input[i])` **plus `jumbled_input[i-1]`**, adds `'a'` to every byte, and (if length == 100) `strncmp`s against `"mlaebfkoibhoijfidblechbgg..."`. On success: "You got the key, congrats! Now xor it with the flag!"

## 2. Tools
- gdb + Python API (the oracle)
- Python (XOR the two hex strings)

## 3. Recon
1. The autoregressive structure is the key observation: `jumbled_input[i] = f(user_input[i], jumbled_input[i-1])`. If you know the correct prefix, position `i` depends only on candidate `user_input[i]` — a per-position oracle with 16 options.
2. Find the observation point: right after the `'a'`-addition loop, at the `cmp dword [n], 0x64` instruction (e.g. `0x55555555499d`), the buffer `jumbled_input` at `rbp-0x70` holds the final jumbled characters.

**[TODO]** screenshot: the Ghidra decompile of the jumble + comparison logic.

## 4. Solution Path
1. **GDB oracle script** — nearly identical to Easy as GDB (writeup 25), reading the buffer instead of flags:
```python
import gdb
from queue import Queue
ALPHABET = "0123456789abcdef"

class Checkpoint(gdb.Breakpoint):
    def __init__(self, queue, *args):
        super().__init__(*args); self.silent = True; self.queue = queue
    def stop(self):
        index = self.queue.get(timeout=10)
        val = gdb.parse_and_eval("((char*)($rbp - 0x70))[{}]".format(index))
        self.queue.put(chr(int(val) & 0xFF))
        return False

bp = Checkpoint(Queue(), "*0x55555555499d")
known = ""
for i in range(100):
    for c in ALPHABET:
        candidate = known + c + "0" * (99 - i)
        gdb.execute("run <<< {}".format(candidate))
        produced = queue.get(timeout=10)
        if produced == EXPECTED[i]:
            known += c
            break
    gdb.execute("kill")
```
(`EXPECTED` is the hardcoded `"mlaebfkoibhoijfidblechbggcgldicegjbkcmolhdjihgmmieabohpdhjnciacbjjcnpcfaopigkpdfnoaknjlnlaohboimombk"`.) The trailing zeros don't matter: autoregression means positions beyond `i` never influence earlier output.
2. **XOR the key with the provided hex** once the binary congratulates you:
```python
a = bytes.fromhex(key_hex)
b = bytes.fromhex(open("hexfile.txt").read().strip())
print(bytes(x ^ y for x, y in zip(a, b)).decode())
```
(Or simply let the binary itself print the congrats and do the XOR by hand — 100 bytes each.)

Why it works: the jumble's dependence chain runs strictly left-to-right, so fixing a prefix isolates each next character. 16 candidates x 100 positions is 1600 runs — seconds of gdb.

**[TODO]** evidence: the recovered key + the XOR output.

## 5. Commands / Payloads Used
```python
((char*)($rbp - 0x70))[i]                      # read jumbled char i
key ^ provided_hex                             # final step
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The binary validated a 100-character hexametric key through an autoregressive transformation compared against a hardcoded target (<evidence>). A GDB Python oracle enumerated the 16-character alphabet per position, exploiting the left-to-right dependence chain to recover the full key, which was then XORed with the provided ciphertext to produce the flag. Per-position validation leaks secrets regardless of transform complexity; security decisions must not expose intermediate state observable in a debugged process."

## 8. Lessons
- Autoregressive checks are brute-forceable position by position — recognize `f(x[i], out[i-1])` patterns in decompiled loops.
- Fill unknown suffix with anything: later positions can't affect earlier outputs when the dependency flows one way.
- This + Easy as GDB + not crypto form a trilogy: the debugger reads what the algorithm hides. Master one template, solve all three.
