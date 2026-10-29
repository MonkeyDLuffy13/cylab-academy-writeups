# CyLab Academy — Vault Door 8 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | java, bitwise ops, brute force per char |
| Status | Solved |

> **Goal:** series finale: nested bitwise transformations (shifts, ORs, masks) per character. Instead of inverting the math by hand, exploit its structure — each character is transformed independently, so brute-force each one against the check.

---

## 1. Understanding the challenge
`VaultDoor8.java` compares each expected character against a computed expression:
```java
boolean check(char c) {
    byte x = (byte) c;
    x = (byte) ((x >>> 1) & 0x??);   // shifts and masks, chained
    x = (byte) (x |  0x??);
    x = (byte) (x ^  0x??);
    // ...
    return x == 0x??;                 // per-character target
}
```
Each character flows through the same chain of bitwise ops and must land on a hardcoded constant.

## 2. Tools
- Python (brute-force loop)
- Text editor

## 3. Recon
Transcribe from the source: the op chain (same for all characters or per-index variants) and the 32 target constants. Note whether ops are index-dependent — most vault-door-8 builds use one shared chain.

**[TODO]** screenshot: the op chain + targets.

## 4. Solution Path
1. Do not invert the chain — brute-force each character through it:
```python
def transform(x, index):
    x = (x >> 1) & 0x??               # transcribe the EXACT chain, in order,
    x = (x |  0x??) & 0xFF            # including Java's byte truncation (& 0xFF)
    x = (x ^  0x??) & 0xFF
    return x & 0xFF

targets = [0x??, 0x??, ...]           # the 32 expected constants

flag_body = ""
for i, t in enumerate(targets):
    for c in range(32, 127):          # printable ASCII only
        if transform(c, i) == t:
            flag_body += chr(c)
            break

print("picoCTF{" + flag_body + "}")
```
2. Gotcha that saves an hour: Java `byte` is signed and truncates to 8 bits — mask every intermediate with `& 0xFF` in Python or your results drift.
3. Verify against the original source logic; every character must pass.

Why it works: per-character, bijective transformations with a printable-only input space give a search space of ~95 per position — brute force is instant and inversion-proof by construction.

**[TODO]** evidence: the script output matching all 32 targets.

## 5. Commands / Payloads Used
```python
for c in range(32, 127): if chain(c) == target: keep   # per-position brute force
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The verifier applies deterministic per-character bitwise transformations to input and compares against fixed constants (<evidence>). Because each character is evaluated independently over a small printable domain, the expected secret is recoverable by exhaustive per-position search — no algebraic inversion required. Remediation: client-side verifiers cannot protect secrets; server-side authentication with salted adaptive hashes (bcrypt/argon2) is the only durable fix."

## 8. Lessons
- When a transform is per-character and bijective, brute-force beats inversion — 95 candidates per position is nothing.
- Porting Java bit ops to Python: always `& 0xFF` after each step (signed byte truncation is the #1 porting bug).
- The vault-door series complete (1: index shuffle, 7: byte reversal, 8: bitwise chains) — one workflow: transcribe, mirror, verify.
