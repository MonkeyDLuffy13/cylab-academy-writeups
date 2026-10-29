# CyLab Academy — Droids 4 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2019) |
| Tags | android, AES, crypto reversing, frida |
| Status | Solved |

> **Goal:** droids finale: the flag ships *inside* the app but encrypted. Recover the key and algorithm from the code, then decrypt — or hook the decrypting function at runtime with Frida and let the app decrypt it for you.

---

## 1. Understanding the challenge
`droids4.apk` carries an encrypted blob (in resources/strings or code) plus the decryption routine. Correct input triggers decryption of the real flag. Two viable routes: reimplement the crypto offline, or intercept it live.

## 2. Tools
- jadx (locate key + ciphertext + algorithm)
- Python pycryptodome (offline decrypt) **or** Frida (runtime hook)
- adb/emulator

## 3. Recon
```bash
jadx -d out droids4.apk
grep -rn "Cipher\|SecretKey\|AES\|decrypt\|pico" out/sources/<package>/ | head -20
```
Typical finding shape:
```java
SecretKeySpec key = new SecretKeySpec("<key>".getBytes(), "AES");
Cipher c = Cipher.getInstance("AES/ECB/PKCS5Padding");
c.init(Cipher.DECRYPT_MODE, key);
String flag = new String(c.doFinal(<encrypted-bytes>));
```
**[TODO]** screenshot: the crypto block in jadx.

## 4. Solution Path
**Route A — reimplement offline:**
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import base64

key = b"<key-from-source>"                 # 16/24/32 bytes
ct  = base64.b64decode("<blob-from-source>")
pt  = unpad(AES.new(key, AES.MODE_ECB).decrypt(ct), 16)
print(pt)
```
(Match the mode/padding string from the source exactly — ECB vs CBC changes the call, not the principle.)

**Route B — hook it live (the two-line Frida win):**
```javascript
Java.perform(function () {
    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.doFinal.overload("[B").implementation = function (b) {
        var out = this.doFinal(b);
        console.log("decrypted: " + bytesToString(out));   // flag lands here
        return out;
    };
});
```
```bash
frida -U -f <package> -l hook.js
```
Trigger the input path in the app; the hook prints the decrypted flag regardless of key strength.

Why it works: the app must hold both key and ciphertext to function — so both are extractable. And it must be *able* to decrypt, so hooking the crypto API makes the app betray its own secret. Crypto in a fully attacker-controlled client is access control, not protection.

**[TODO]** evidence: jadx crypto block + decrypted output (script or hook).

## 5. Commands / Payloads Used
```python
AES.new(key, AES.MODE_ECB).decrypt(ct)     # mirror the source's transformation
```
```javascript
Java.use("javax.crypto.Cipher").doFinal.overload("[B").implementation = ...
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The application ships an encrypted secret together with its decryption key and routine (<evidence>). Both static reimplementation and runtime API hooking recovered the plaintext. Local cryptography cannot enforce entitlements on untrusted devices — the same class of finding as hardcoded mobile API credentials. Remediation: secrets the client must not know should not be on the client; fetch them from an authenticated server; use mobile attestation only as risk-scoring friction."

## 8. Lessons
- Search terms that find mobile crypto instantly: `Cipher`, `SecretKey`, `AES/`, `doFinal`, `IvParameterSpec`.
- Frida's `Cipher.doFinal` hook is the single highest-value mobile script you will write — it beats reimplementing every custom crypto routine.
- The droids series complete: logcat -> jadx -> native -> smali patch -> crypto. It is the compressed version of an entire Android pentest methodology.
