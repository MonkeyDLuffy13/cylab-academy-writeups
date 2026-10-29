# CyLab Academy — Safe Opener · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Points | 100 |
| Tags | java, base64, static analysis |
| Status | Solved |

> **Goal:** first RE read — open a small Java program, find where the "password" lives, and recognize an encoding instead of an encryption. Base64 disguises strings; it never protects them.

---

## 1. Understanding the challenge
A Java source file (`SafeOpener.java`) implementing a safe that "opens" when given the right key. The code stores the expected key encoded, and validates your input against it.

## 2. Tools
- Text editor (the source is provided)
- `base64` / CyberChef
- `javac`/`java` (optional, to run it)

## 3. Recon
Read the source and locate the check:
```java
// inside openSafe(): 
// encodedkey = Base64.getEncoder().encode(key.getBytes()) is NOT here —
// instead the stored constant is compared after Base64.getDecoder().decode(...)
String decoded = new String(Base64.getDecoder().decode("<STORED_B64>"));
return decoded.equals(input);
```
The stored constant is a Base64 blob (telltale: `picoCTF` would appear as something like `cGljb0NURns...` — Base64 of `picoCTF{` is literally the prefix `cGljb0NURnt`).

**[TODO]** screenshot: the relevant lines of `SafeOpener.java`.

## 4. Solution Path
```bash
echo '<STORED_B64>' | base64 -d
```
The decoded string is the flag (or the password that makes `java SafeOpener` print "flag opened").

Why it works: Base64 is a transport encoding with no key. Anyone who can read the constant can decode it — the code itself contains the entire "secret". This is the same finding class as hardcoded credentials in mobile apps (my Android/iOS writeups hit it constantly with `jadx` + `strings`).

**[TODO]** evidence: the `base64 -d` output.

## 5. Commands / Payloads Used
```bash
echo '<b64>' | base64 -d
# or CyberChef: From Base64
# or python3 -c "import base64;print(base64.b64decode('<b64>').decode())"
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "Static review of the client application revealed the access secret embedded as a Base64-encoded constant (<evidence>). Because Base64 provides encoding rather than confidentiality, the secret was recovered without execution or key material. Remediation: remove secrets from client-side code entirely; validate authorization server-side; and add static-analysis checks for encoded-string patterns (`cGljb0`, `picoCTF`, high-entropy literals) to release pipelines."

## 8. Lessons
- Spot Base64 on sight: `picoCTF{` -> `cGljb0NURns`, `CTF` variants, and strings ending in `=` (optional padding).
- RE order of effort: read the source/strings first; reach for decompilers and debuggers only when logic actually needs them.
- Same lesson as mobile RE: whatever the client checks, the client leaks.
