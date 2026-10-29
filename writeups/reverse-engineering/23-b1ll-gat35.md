# CyLab Academy — B1ll_Gat35 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | windows, x64dbg, aslr, memory inspection |
| Status | Solved |

> **Goal:** a Windows binary generates a key at runtime, shows it to you in memory, then asks you to type it back. The twist: the expected input is not the key — it is the entire "The key is: ..." string, prefix included.

---

## 1. Understanding the challenge
"Can you reverse this Windows Binary?" — a 32-bit Windows executable. Flow: it asks for a number (1-5 digits), "initializes" a key derived from your input, then demands the correct key. Ghidra shows `initialize_key(...)` and `validate_key(...)` — but the validation internals are tedious to read statically. This challenge is designed for a debugger.

## 2. Tools
- **x64dbg** (debugging)
- **CFF Explorer** (disable ASLR in the PE header)
- Ghidra (orientation only)

## 3. Recon
1. Run it:
```
Input a number between 1 and 5 digits: 1
Initializing...
Enter the correct key to get the access codes: test
Incorrect key. Try again.
```
2. Ghidra `main()`: counts digits of your number, passes (last digit, digit count) to `initialize_key`, reads your key, calls `validate_key`. The check functions are deeply nested — dynamic analysis is faster.
3. **Kill ASLR first**: open the binary in CFF Explorer -> NT Headers -> Optional Headers -> DllCharacteristics -> uncheck "DLL can move" -> save. Now addresses are stable between runs and you can rebase Ghidra to match.

**[TODO]** screenshot: the Ghidra decompile of `main`.

## 4. Solution Path
1. In x64dbg, step into `initialize_key`. After it returns, a string is sitting in a register:
```
eax: "The key is: 425"
```
2. Try `425` as the key — **"Incorrect key."** The program wants something longer.
3. Continue stepping to where your typed key is processed. Before `validate_key`, another buffer appears:
```
ecx: "The key is: 4253360"
```
4. Try `4253360` — still "Incorrect key." The comparison target is the **whole formatted string**, prefix and all. Type it exactly as seen in memory:
```
Enter the correct key to get the access codes: The key is: 4253360
Correct input. Printing flag: PICOCTF{These are the access codes to the vault: 1063340}
```
5. Lesson confirmed: the developer compared your input against the full sprintf output, not the number. The debugger showed it; nothing needed reversing.

Why it works: any value constructed at runtime exists in process memory right before it is used. Stepping through the two calls that touch your input exposes the expected string directly — no need to understand the key derivation math.

**[TODO]** evidence: the x64dbg register dumps + the flag screen.

## 5. Commands / Payloads Used
```
x64dbg: step into initialize_key / validate_key, inspect registers
key input: The key is: 4253360
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The Windows binary validated a runtime-generated access code by comparing user input against a formatted string held in process memory (<evidence>). With ASLR disabled, a debugger exposed the expected value in a register immediately before the comparison, and the literal string was accepted as input. Secret validation that depends on values present in a debuggable process is recoverable; hardened clients should avoid all-plaintext comparisons and assume full attacker control of the execution environment."

## 8. Lessons
- "Read the string in the debugger" beats "reverse the algorithm" almost every time — try the cheap route first.
- When a recovered value fails, look at its *context* in memory: prefixes, labels, and formatting are often part of the expected input.
- CFF Explorer's "DLL can move" toggle is the standard pre-debugging ritual for old 32-bit Windows binaries.
