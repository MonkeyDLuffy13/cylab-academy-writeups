# CyLab Academy — Droids 1 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | android, jadx, static analysis, dex |
| Status | Solved |

> **Goal:** droids 1 moves the secret into the code: the app validates an input password and the flag is bound to it. Decompile the DEX with jadx and read the check — the workflow that anchors all static Android work.

---

## 1. Understanding the challenge
`droids1.apk` asks for a password/flag input. The comparison lives in the app's Java/Kotlin code (compiled to DEX). No native library involvement yet — the answer is one decompile away.

## 2. Tools
- `jadx` / `jadx-gui`
- `apktool` (alternative: smali view)
- Emulator + adb (verify)

## 3. Recon
```bash
jadx -d out droids1.apk        # or jadx-gui for browsing
```
Navigate to the main package -> MainActivity (or wherever `onCreate`/button handler lives). Look for:
```java
if (inputText.getText().toString().equals("<something>")) { /* flag path */ }
```
or a hash/format check referencing the flag string.

**[TODO]** screenshot: the decompiled check in jadx.

## 4. Solution Path
1. Read the check in the decompiled source. Variants you may meet in this series:
   - the expected string *is* the flag (just read it),
   - the expected string is a transformation of the flag (apply the inverse),
   - the check compares hashes (crack/verify offline).
2. This level: the correct input is recovered directly from the constant in the check (possibly per-character assembled — same extraction as vault-door-1 if so).
3. Verify dynamically: install, type the recovered string, app prints the flag:
```bash
adb install droids1.apk    # then interact, or watch logcat for the success branch
adb logcat | grep -i pico
```

Why it works: DEX bytecode decompiles to near-source Java — constants, comparisons, and string operations all survive. Anything the app "knows" is in the APK, and the APK ships to every user.

**[TODO]** evidence: the extracted expected string + the app accepting it.

## 5. Commands / Payloads Used
```bash
jadx -d out droids1.apk
grep -rn "pico\|flag\|equals" out/sources/<package>/ | head
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "Static decompilation of the application revealed the authentication check including its embedded expected secret (<evidence>), allowing complete recovery of the credential without execution. Client-side credential verification is equivalent to credential disclosure (OWASP MASVS-AUTH finding). Remediation: authenticate server-side, and treat any secret in client code — string, encoded, or computed — as public."

## 8. Lessons
- jadx reflex: `jadx -d out app.apk` then `grep -rn "equals\|pico\|flag" out/sources/`. Two commands, most of the answer.
- If jadx output is obfuscated (single-letter classes), search for what stays unique: strings, resources, native method names.
- Cross-link: the full tooling and checklist for this flow lives in my android-pentesting-cheatsheet repo.
