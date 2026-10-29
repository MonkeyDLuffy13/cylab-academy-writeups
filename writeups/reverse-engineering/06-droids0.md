# CyLab Academy — Droids 0 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | android, apk, logcat, dynamic analysis |
| Status | Solved |

> **Goal:** open the droids series — five Android APKs. Droids 0 needs zero decompilation: install the app, run it, and read the flag straight out of the device log. If you did my Android labs, this is a warm-up lap.

---

## 1. Understanding the challenge
`droids0.apk` is a small app with a button. The challenge description points at logging. The app writes the flag to the system log when the button is pressed — visible to anyone (or any app) reading `logcat`.

## 2. Tools
- Android emulator or device (adb available)
- `adb install`, `adb logcat`
- Optional: Android Studio Logcat panel

## 3. Recon
```bash
adb install droids0.apk
adb logcat -c                 # clear the buffer first — signal, not noise
```
Launch the app (emulator GUI or `adb shell monkey -p <package> 1`).

**[TODO]** screenshot: the app on the emulator.

## 4. Solution Path
1. Start a live log stream filtered to the app:
```bash
adb logcat | grep -iE 'flag|pico'
```
2. Press the app's button.
3. The flag appears in the log line.

Why it works: Android's logging is system-global and (historically) readable by any app with the log permission — writing secrets to `Log.d()` is one of the most common real-world Android leaks. This is the first checklist item of every Android triage: *run it and watch logcat* — exactly the first step in my android-pentesting-cheatsheet triage flow.

**[TODO]** evidence: the logcat line containing the flag.

## 5. Commands / Payloads Used
```bash
adb install droids0.apk
adb logcat -c && adb logcat | grep -iE 'pico|flag'
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The application logs sensitive data (the flag material) to the system log at runtime (<evidence>). On untrusted or multi-tenant devices, logcat output is readable by other applications and by anyone with adb access, constituting direct secret disclosure (OWASP MASVS-RESILIENCE/logging findings). Remediation: strip all secret and debug logging from release builds (ProGuard/R8 assumption removal, `Log` call elision), and never log user data or credentials."

## 8. Lessons
- Dynamic triage before static: install -> clear logcat -> interact -> grep. It solves the easy half of every droid challenge.
- `logcat -c` matters: stale buffer lines masquerade as your signal.
- droids series roadmap: 0 logcat -> 1 jadx static -> 2 native lib -> 3 smali patch -> 4 crypto reversing. Each adds one tool to the mobile kit.
