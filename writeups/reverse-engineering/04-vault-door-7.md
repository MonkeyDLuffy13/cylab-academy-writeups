# CyLab Academy — Vault Door 7 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | java, endianness, Integer.reverseBytes |
| Status | Solved |

> **Goal:** vault-door series, round 7: the expected password is stored as `int`s whose bytes are *word-reversed* (`Integer.reverseBytes`). Spot the endianness joke, invert it with `struct`, done.

---

## 1. Understanding the challenge
`VaultDoor7.java` holds the expected password as an `int[]`:
```java
int[] myBytes = { 0x??, 0x??, ... };
StringBuffer out = new StringBuffer();
for (int b : myBytes) out.append((char) Integer.reverseBytes(b));
// compared against your input
```
Each int packs four characters, but the bytes arrive in reverse order inside every 32-bit word.

## 2. Tools
- Python `struct` (byte order control)
- Text editor

## 3. Recon
Read the source and confirm: (a) the transformation applied to the ints before comparison is `Integer.reverseBytes`, (b) the raw int values in the array. Nothing else is hidden — the whole secret is in the literals.

**[TODO]** screenshot: the int array + the reverseBytes loop.

## 4. Solution Path
Rebuild the password by applying the same word-reversal to each literal:
```python
import struct

ints = [0x3c7a8d8b, 0x??, ...]                  # transcribe the array

out = b"".join(struct.pack(">i", i)[::-1] for i in ints)
# equivalent: struct.pack("<i", i) for each  -> little-endian already reverses
out2 = b"".join(struct.pack("<I", (i & 0xFFFFFFFF)) for i in ints)
print(out.hex(), out2)
password = "".join(chr(b) for b in out2 if 32 <= b < 127)
print("picoCTF{" + password + "}")
```
Either endianness works as long as you reverse the four bytes of each word; verify by checking the first int decodes to `pico` (the flag prefix anchors always show up in word 1).

Why it works: `Integer.reverseBytes` is a bijection on 32-bit values — applying it again returns the original. "Obfuscation" that is its own inverse is just an encoding.

**[TODO]** evidence: the decoded string and (optionally) `java VaultDoor7` accepting it.

## 5. Commands / Payloads Used
```python
struct.pack("<I", word)     # little-endian = the reversal, for free
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The verifier derived its expected secret from word-reversed integer constants (<evidence>). The transformation is self-inverse and the constants ship in the artifact, so the secret is recoverable by direct application of the same transformation — no cryptanalysis required. Remediation: secrets must not exist, in any encoding, inside client-verifiable artifacts; move authentication server-side."

## 8. Lessons
- Java ints in flag challenges usually pack 4 ASCII chars per word — decode in groups of 4.
- `struct.pack('<I', x)` silently does the endianness work that `reverseBytes` does — know your byte orders (`<` little, `>` big).
- Self-inverse ops seen so far in the series: ROT13/XOR (crypto), reverseBytes (here). Spot them and the "encryption" evaporates.
