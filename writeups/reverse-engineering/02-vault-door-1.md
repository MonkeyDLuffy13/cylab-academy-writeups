# CyLab Academy — Vault Door 1 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Points | 100 |
| Tags | java, ASCII, source-audit |
| Status | Solved |

> **Goal:** start the `vault-door` training series — read a Java password check and reverse the transformation with a small script. This first vault hides the password as ASCII codes in a shuffled order.

---

## 1. Understanding the challenge
`VaultDoor1.java` contains `checkPassword(...)`: it compares individual characters of your input to hardcoded integer ASCII values at specific indexes. The expected password never appears as a string — only as ~32 numeric literals with `charAt(i)` lookups.

## 2. Tools
- Text editor
- Python (rebuild the password from the constraints)

## 3. Recon
The check body looks like:
```java
return (
    password.charAt(0)  == 'd' &&
    password.charAt(29) == '3' &&
    password.charAt(1)  == 'c' &&
    // ... ~32 constraints mixing constants and indexes
);
```
Some vault-door levels use ASCII ints (`password.charAt(3) == 99`). Read both forms; the solve script is the same after `chr()`.

**[TODO]** screenshot: the constraint block.

## 4. Solution Path
Transcribe the constraints into a dict index -> char, then rebuild in order:
```python
expected = {
    0:'d', 29:'3', 1:'c', 4:'0', 5:'m', 30:'1', 9:'b', 6:'_',
    2:'0', 27:'8', 8:'4', 26:'a', 13:'y', 10:'_', 31:'c',
    3:'8', 7:'r', 25:'_', 14:'0', 12:'u', 11:'r', 15:'n',
    24:'b', 23:'f', 16:'3', 22:'_', 17:'d', 21:'v', 20:'3',
    18:'1', 19:'m', 28:'e'
    # <-- transcribe YOUR challenge's exact pairs
}
flag_body = "".join(expected[i] for i in sorted(expected))
print("picoCTF{" + flag_body + "}")
```
Notes that save time:
- The wrapper is literally `picoCTF{...}` — the vault checks only the inner 32 chars (0..29 plus the two trailing chars at 30/31 in some levels); read `checkPassword`'s expected length and format comment (`picoCTF{` + 32 chars + `}`).
- If the constraints use integers, build the dict with `chr(value)`.

Verify against the original source by mentally running `checkPassword(your_flag)` — every branch must pass.

**[TODO]** evidence: script output and (optionally) `java VaultDoor1` accepting the input.

## 5. Commands / Payloads Used
```python
"".join(expected[i] for i in sorted(expected))     # constraint dict -> password
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The password verifier embedded its full expected secret as a set of per-character constraints (<evidence>). Constraint-based checks are logically equivalent to storing the secret: the satisfying assignment reconstructs it completely, here via a constraint dictionary. Remediation: secrets must never be validated client-side; move verification to the server and store only salted adaptive hashes (bcrypt/argon2) of authentication material."

## 8. Lessons
- Any "check" code that references the answer per-character hands you the answer; extract, sort, join.
- Solve constraints with the simplest tool that works — a dict beats Z3 for direct mappings (Z3 becomes the right tool when constraints are arithmetic, e.g., vault-door-3+).
- The vault-door series escalates: expect byte arrays, XOR, shifts, and permutation tricks — same workflow every time.
