# CyLab Academy — Powershelly · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2021) |
| Tags | powershell, base64, gzip, script decoding |
| Status | Solved |

> **Goal:** unwrap an encoded PowerShell payload. `-EncodedCommand` blobs are base64 of (usually gzip-compressed) UTF-16LE script — the same wrapping real phishing/aspera-style malware uses. Decode it layer by layer and read what it actually does.

---

## 1. Understanding the challenge
The challenge ships a PowerShell invocation (a `.ps1` or a one-liner) with a long base64 blob — the classic `powershell -e <b64>` pattern. The flag logic lives inside the compressed script.

## 2. Tools
- Python (base64 + gzip decode)
- `strings` on the blob (sometimes a shortcut)
- A PowerShell prompt (optional, to run the decoded script)

## 3. Recon
```bash
echo '<b64-blob>' | base64 -d | strings | head
```
If it prints readable PowerShell — done early. If it prints binary noise: the blob is compressed (gzip/deflate), which is the common case and the reason the challenge exists.

**[TODO]** screenshot: the blob and its source invocation.

## 4. Solution Path
1. Decode base64 -> gunzip -> read:
```python
import base64, gzip

blob = open('encoded.txt','rb').read().strip()
raw  = base64.b64decode(blob)

try:
    script = gzip.decompress(raw).decode('utf-16-le')   # the standard wrap
except Exception:
    import zlib
    script = zlib.decompress(raw, -15).decode('utf-16-le')   # raw deflate fallback

print(script)
open('decoded.ps1','w',encoding='utf-8').write(script)
```
The decompression wrapper note: `-EncodedCommand` expects base64(UTF-16LE(script)); challenge blobs additionally gzip first — the decoder chain is base64 -> gunzip -> utf-16le.
2. Read `decoded.ps1`: the flag logic is visible — typically a character array / arithmetic construction of the flag string, or a loop comparing input.
3. If it computes rather than contains: run the transformed-array logic in Python (transcribe, as in every vault-door-style challenge), or execute the script in a scratch PowerShell and let it print.

Why it works: encoding + compression are transport layers, not protection. Every "encoded PowerShell attack" works this way — which is exactly why defenders decode-and-detonate in sandboxes and why AMSI inspects scripts *after* decompression.

**[TODO]** evidence: decoded script + the recovered flag.

## 5. Commands / Payloads Used
```python
base64.b64decode -> gzip.decompress -> .decode('utf-16-le')
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The analyzed script obfuscated its logic using base64 encoding and compression (<evidence>), fully recoverable through layered decoding without any secret material. Encoded payloads should be treated as opaque containers: automated decode-and-inspect pipelines (and runtime script inspection such as AMSI) are required controls, since static encoding provides no confidentiality against defenders or analysts."

## 8. Lessons
- The decoder chain to memorize for `powershell -e`: base64 -> (gunzip, if present) -> UTF-16LE. Ninety percent of encoded payloads open with exactly this key.
- When compressed content resists, check magic bytes: `1f 8b` gzip, `78 9c` zlib — decode accordingly.
- This is the defensive mirror of offensive tradecraft: the same decode chain you will use in phishing-attachment triage and threat-hunt scripts.
