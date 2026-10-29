# CyLab Academy — Mind your Ps and Qs · Writeup

| | |
|---|---|
| Category | Cryptography |
| Points | 250 |
| Tags | RSA, small exponent, CRT, parameter hygiene |
| Status | Solved |

> **Goal:** read RSA public parameters (`n`, `e`, `c`) out of PEM files and exploit the classic "small e, small message" weakness — the first genuinely modern-crypto break in the library.

---

## 1. Understanding the challenge
You are given several outputs of RSA encryption, each with its own modulus: same small public exponent `e = 3`, different `n_i`, same message `m` (the flag, a short number after PKCS-less encoding). Because `m^3` for a short message is smaller than some of the moduli, those ciphertexts are not actually reduced — and with e=3 and two moduli sharing a factor (or via Hastad/CRT), the message falls out.

Two practical attack routes:

- **Shared prime:** if any pair of moduli `n1, n2` shares a prime, `gcd(n1, n2)` recovers it -> factor both -> decrypt.
- **Small message, e=3:** if `m^3 < n`, then `c = m^3` and the plaintext is a plain integer cube root.

## 2. Tools
- Python 3 + `pycryptodome` (or `sympy`)
- `openssl rsa -text` to read PEM parameters
- RsaCtfTool (automation cross-check)

## 3. Recon
```bash
openssl rsa -pubin -in key1.pub -text -noout    # prints n and e
python3 - <<'EOF'
from Crypto.PublicKey import RSA
for f in ["key1.pub","key2.pub","key3.pub"]:
    k = RSA.import_key(open(f,'rb').read())
    print(f, "e =", k.e, "n bits =", k.n.bit_length())
EOF
```
**[TODO]** screenshot: the printed parameters; note which exponent/moduli pair looks suspicious.

## 4. Solution Path
Route A — shared factor between two public keys:
```python
from math import gcd
from Crypto.Util.number import long_to_bytes

n1, n2 = ..., ...                 # from the pubkeys
c1, c2 = ..., ...                 # matching ciphertexts
p = gcd(n1, n2)
q1, q2 = n1 // p, n2 // p
d1 = pow(65537 if False else 3, -1, (p-1)*(q1-1))
m = pow(c1, d1, n1)
print(long_to_bytes(m))
```
Route B — no reduction happened (small message, e=3):
```python
import gmpy2
from Crypto.Util.number import long_to_bytes
m, exact = gmpy2.iroot(c, 3)
print(exact, long_to_bytes(int(m)))
```
Cross-check automatically with RsaCtfTool:
```bash
python3 RsaCtfTool.py --publickey key1.pub --uncipherfile cipher1 --attack gcd,hastads
```

**[TODO]** evidence: recovered plaintext bytes showing the flag.

## 5. Commands / Payloads Used
```text
gcd(n1, n2)                    # shared-prime kill shot
iroot(c, e)                    # unreduced small-message case
openssl rsa -pubin -text       # always read the parameters first
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The reviewed deployment reused key material across RSA keypairs such that two public moduli shared a prime factor. Computing gcd(n1, n2) factored both keys without a factoring effort (<evidence>), enabling plaintext recovery. Remediation: generate moduli from a CSPRNG, never reuse primes, prefer e = 65537 with proper padding (OAEP), and add pairwise gcd audits to key-generation pipelines (the ROCA lesson)."

## 8. Lessons
- Before attacking RSA: inventory `e`, `n` bit size, and whether padding is mentioned. The weakness is almost always in the parameters.
- `gcd` across collected public keys costs nothing and finds industrial-scale key-generation failures (Debian, ROCA).
- `long_to_bytes` converts recovered integers to readable flag strings — keep it imported at all times.
