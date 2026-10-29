# CyLab Academy — Droids 3 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | android, smali patching, apktool, bypass |
| Status | Solved |

> **Goal:** droids 3 flips the objective: stop *reading* the secret, start *bypassing* the check. Decompile to smali, flip the branch, repackage, sign, install. This is the fundamental Android patch-and-resign loop.

---

## 1. Understanding the challenge
`droids3.apk` validates an input against a check that is not practically reversible (hash or scattered comparison in the Java/native layer). You do not need the original password — you need the app's success branch to fire regardless of input.

## 2. Tools
- `apktool` (decode/build)
- `uber-apk-signer` or `jarsigner`+`zipalign` (re-sign)
- smali/baksmali understanding (branch opcodes)
- adb + emulator

## 3. Recon
```bash
apktool d droids3.apk -o droids3
grep -rn "check\|equals\|pico" droids3/smali/<package>/ | head
```
Find the check and the success branch. In smali, the shape is:
```smali
invoke-...checkFlag(...)Z
move-result v0
if-eqz v0, :fail_label          # <- the branch to flip
...success path (prints flag)...
```
**[TODO]** screenshot: the check smali with the if-eqz highlighted.

## 4. Solution Path
1. Flip the branch: change `if-eqz v0, :fail` to `if-nez v0, :fail` — or simpler and more robust: replace the conditional with an unconditional jump:
```smali
goto :success_label             # skip the check entirely
```
2. If the check result feeds a toast/text rather than a branch, instead force the register: replace `move-result v0` with `const/4 v0, 0x1`.
3. Rebuild, sign, install:
```bash
apktool b droids3 -o droids3-patched.apk
uber-apk-signer -a droids3-patched.apk      # produces *-aligned-debugSigned.apk
adb install droids3-patched-aligned-debugSigned.apk
```
4. Run it, enter anything, and the patched branch fires — the flag appears (on screen or in logcat).

Why it works: the decision "is this input correct?" is a byte in the app's own bytecode. Whoever controls the APK controls the decision. Repackaging is only gated by the signature — and the signature is yours.

**[TODO]** evidence: the patched smali diff + the app revealing the flag on arbitrary input.

## 5. Commands / Payloads Used
```bash
apktool d / apktool b
if-eqz -> goto            # or move-result -> const/4 v0, 1
uber-apk-signer -a out.apk
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The application's license/credential decision is enforced in smali code branchable by the attacker (<evidence>). Decompiling, inverting the verification branch, rebuilding, and re-signing the package restored full functionality without the valid credential. Remediation: no client-side integrity mechanism survives a controlled device; enforce licensing and entitlements server-side per request, and apply attestation (Play Integrity) as friction — not as the control."

## 8. Lessons
- The patch loop to memorize: `apktool d` -> edit smali -> `apktool b` -> sign -> install. It is the backbone of Android CTFs and of real-world cracked-app analysis.
- Smali branch pairs: `if-eqz`/`if-nez` are opposites — flipping one character often flips the logic.
- droids4 (next) keeps the patch loop but adds crypto — the flag itself is encrypted in resources.
