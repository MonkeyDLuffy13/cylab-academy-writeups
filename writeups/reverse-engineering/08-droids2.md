# CyLab Academy — Droids 2 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | android, native library, JNI, strings |
| Status | Solved |

> **Goal:** droids 2 hides the logic in a native `.so` library via JNI. The Java side is a decoy — the secret lives in the binary. Cross the Java/native boundary and read it.

---

## 1. Understanding the challenge
`droids2.apk` looks like droids 1 (an input check), but the decompiled Java calls into a native method (`native boolean checkFlag(...)` style). The comparison happens inside `lib/<abi>/*.so`.

## 2. Tools
- `unzip` (an APK is a zip)
- `strings`, `readelf`
- Ghidra/IDA (if the check is not string-visible)
- adb/emulator to verify

## 3. Recon
```bash
unzip -o droids2.apk -d droids2
ls droids2/lib/*/                    # e.g. lib/arm64-v8a/libhello-jniLib.so
strings -a droids2/lib/*/lib*.so | grep -iE 'pico|flag'
```
**[TODO]** screenshot: the `lib/` contents + the strings hit.

## 4. Solution Path
1. The `strings` sweep usually lands the flag directly (JNI challenge binaries at this level store it as a literal).
2. If the string is assembled or encoded instead:
   - Load the `.so` in Ghidra, find the exported JNI function (`Java_<package>_<activity>_checkFlag` naming), read its logic.
   - Look for XOR tables / char arrays near the function and decode as in the vault-door writeups.
3. Verify: enter the recovered flag in the app (or watch the success log branch).

Why it works: moving code native raises decompilation effort (that is its real purpose in app hardening) but moves nothing out of reach — the library is still a file you own, and `strings` remains the cheapest tool in the box before any disassembler work.

**[TODO]** evidence: flag string from the library (or decoded constant).

## 5. Commands / Payloads Used
```bash
unzip -o droids2.apk -d droids2
strings -a droids2/lib/arm64-v8a/lib*.so | grep -i pico
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The application relocated its credential check into a JNI native library (<evidence>). Static extraction from the shipped library disclosed the embedded secret without execution or memory analysis. Native code raises analysis effort marginally but not fundamentally; shipped binaries remain attacker-inspectable. Remediation: server-side validation of any secret-bearing logic; if native obfuscation is used for IP protection, combine it with the assumption that extraction is always possible eventually."

## 8. Lessons
- APK anatomy: `lib/<abi>/*.so` is where JNI logic hides; always check it before concluding "the Java has no flag".
- Tool escalation for native: `strings` -> `readelf -s` (exports) -> Ghidra on the JNI-named function.
- This is the same native-layer pattern from my Android MHL writeups (Frida hooks on native checks) — the droids series is the beginner ramp to those.
