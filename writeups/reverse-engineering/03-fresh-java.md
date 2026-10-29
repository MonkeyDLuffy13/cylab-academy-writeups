# CyLab Academy — Fresh Java · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Points | 200 |
| Tags | java, decompilation, jadx |
| Status | Solved |

> **Goal:** you only get a `.class` file this time — decompile it, read the key-check logic, and reconstruct the secret. This is the exact workflow used daily in Android app analysis (my mobile writeups run it through `jadx` too).

---

## 1. Understanding the challenge
`KeygenMe.class` (no source). Running it asks for a "key". The check logic lives in bytecode; the intended workflow is decompile -> read -> satisfy the check.

## 2. Tools
- `jadx` (or `jadx-gui`) — bytecode to readable Java
- `javap -c` as a no-install fallback (ships with the JDK)
- Online decompiler as last resort (do not upload real-world/proprietary code there)

## 3. Recon
```bash
file KeygenMe.class                 # compiled Java class data, version X
jadx -d out KeygenMe.class          # emits out/sources/.../*.java
```
Fallback without jadx:
```bash
javap -p -c KeygenMe.class | less   # readable-ish bytecode, strings included
```

**[TODO]** screenshot: decompiled `checkPassword`/`validate` method.

## 4. Solution Path
1. In the decompiled source, locate the validation method. Typical pattern for this challenge family: the expected key is derived from a constant string through simple arithmetic (per-character shifts, XOR with a fixed byte, or index math).
2. Invert the transformation in Python (example shape — transcribe the actual logic you see):
```python
consts = [...]                     # from the decompiled code
key = "".join(chr(c ^ 0x2A) for c in consts)   # e.g. XOR variant
# or: chr(c - i) / chr((c + 13) % 128) — mirror whatever the code does
print(key)
```
3. Run the program and supply the recovered key:
```bash
java KeygenMe    # paste key -> "valid key" + flag
```
If the flag itself is the successful key, the program prints it on acceptance.

**[TODO]** evidence: decompiled check + your inversion script + the program accepting the key.

## 5. Commands / Payloads Used
```bash
jadx -d out KeygenMe.class
javap -p -c KeygenMe.class         # bytecode fallback
java KeygenMe                      # verify the recovered key
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "Analysis of the distributed class file recovered its verification logic via decompilation (<evidence>), and the embedded transformation was inverted to derive the accepted key. Compiled JVM artifacts retain near-source fidelity, so shipping client-side validation leaks its own algorithm — the same property that makes Android APK analysis effective. Remediation: perform secret validation server-side; if obfuscation is required for IP protection, use maintained toolchains (ProGuard/R8) while accepting it raises effort, not guarantees."

## 8. Lessons
- `jadx -d out file.class` (or `.apk`) is one command to readable Java — make it reflexive.
- `javap -c` is the zero-install fallback; strings and constants appear inline in the disassembly.
- Decompile, transcribe the transformation into Python, invert, verify by running the binary. That loop is the backbone of the mobile writeups as well.
