# CyLab Academy — Easy1 (Vigenere) · Writeup

| | |
|---|---|
| Category | Cryptography |
| Points | 100 |
| Tags | vigenere, polyalphabetic, key |
| Status | Solved |

> **Goal:** decrypt a Vigenere cipher with a known key using the tabula recta — and script the table lookup so you never hand-count rows again.

---

## 1. Understanding the challenge
The challenge gives you a ciphertext, a key, and a Vigenere table image. Vigenere shifts each plaintext letter by the letter of the key that lines up with it (repeating the key as needed). It is polyalphabetic: the same plaintext letter encrypts differently depending on position, which defeats naive frequency counting.

## 2. Tools
- CyberChef (Vigenere Decode)
- Python script (below)

## 3. Recon
The relationship from the tabula recta:
```text
cipher[i] = (plain[i] + key[i % len(key)]) mod 26
plain[i]  = (cipher[i] - key[i % len(key)]) mod 26
```
Letters are treated as A=0..Z=25; the key repeats across the message.

## 4. Solution Path
1. Decrypt with CyberChef: recipe **Vigenere Decode**, paste the key into `Key`, ciphertext into `Input`.
2. Or decrypt with a small script (your reusable tool for later challenges):
```python
def vigenere_decrypt(ct: str, key: str) -> str:
    out, k = [], [ord(c) - 65 for c in key.upper()]
    for i, ch in enumerate(ct.upper()):
        if ch.isalpha():
            out.append(chr((ord(ch) - 65 - k[i % len(k)]) % 26 + 65))
        else:
            out.append(ch)
    return "".join(out)

print(vigenere_decrypt("<CIPHERTEXT>", "<KEY>"))
```
The plaintext (readable English) is the flag body; wrap it as the challenge requires.

**[TODO]** evidence: script output / CyberChef screenshot.

## 5. Commands / Payloads Used
```text
plain = (cipher - key) mod 26        # decryption direction
CyberChef recipe: Vigenere Decode
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The ciphertext was encrypted with a Vigenere cipher whose key was disclosed alongside it, reducing the scheme to a deterministic table lookup (<evidence>). More importantly, once the key length is known (or recovered via Kasiski/index-of-coincidence analysis), Vigenere collapses to independent Caesar shifts and is breakable without the key. Conclusion: polyalphabetic classical ciphers are unsuitable for any confidentiality requirement."

## 8. Lessons
- Keep a personal `vigenere.py` — this exact function returns in later labs as a decoder ring.
- Known-plaintext is devastating: if you suspect the prefix `PICOCTF` maps to some ciphertext, the key falls out by subtraction.
- CyberChef recipe order matters: Decode (not Encode) when you have ciphertext in Input.
