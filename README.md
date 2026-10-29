# CyLab Academy Writeups

Step-by-step writeups for the challenge library of the **CyLab Security Academy** ([learn.cylabacademy.org](https://learn.cylabacademy.org/library)) — the free Carnegie Mellon University learning platform formerly known as **picoCTF** (`play.picoctf.org`).

Each writeup follows the same working format used across my other repos: understand the challenge, list the tools, walk the solution path with real commands, capture the evidence, and close with a report-ready impact/remediation paragraph. The goal is to learn the *technique* behind every challenge, not to copy-paste a flag.

> **Disclaimer.** All content here is educational. Every challenge is part of an official, intentionally vulnerable training platform created by security experts at CMU for learning purposes. Nothing in this repo applies to systems the author does not own or is not explicitly authorized to test.

---

## Why this platform

- Free, browser-based, permanently available challenge library (the classic picoGym).
- Challenges are tagged by category and point value, so difficulty scales smoothly from first `cat` to format-string exploitation.
- A built-in webshell means most challenges can be solved with zero local setup.

## Categories

| Category | What it teaches | Writeups |
|---|---|---|
| General Skills | Shell basics, file handling, script arguments | [writeups/general-skills](writeups/general-skills) |
| Cryptography | Classical ciphers, encoding, RSA fundamentals | [writeups/cryptography](writeups/cryptography) |
| Web Exploitation | HTTP methods, source review, cookies, headers | [writeups/web-exploitation](writeups/web-exploitation) |
| Forensics | Metadata, file formats, PDF/SVG internals, data carving | [writeups/forensics](writeups/forensics) |
| Reverse Engineering | Java decompilation, string analysis, key-check logic | [writeups/reverse-engineering](writeups/reverse-engineering) |
| Binary Exploitation | Buffer overflows, format-string bugs | [writeups/binary-exploitation](writeups/binary-exploitation) |

## Writeup index

### General Skills
| # | Challenge | Points | File |
|---|---|---|---|
| 01 | Obedient Cat | 5 | [01-obedient-cat.md](writeups/general-skills/01-obedient-cat.md) |
| 02 | Wave a Flag | 10 | [02-wave-a-flag.md](writeups/general-skills/02-wave-a-flag.md) |
| 03 | Python Wrangling | 10 | [03-python-wrangling.md](writeups/general-skills/03-python-wrangling.md) |
| 04 | Magikarp Ground Mission | 30 | [04-magikarp-ground-mission.md](writeups/general-skills/04-magikarp-ground-mission.md) |

### Cryptography
| # | Challenge | Points | File |
|---|---|---|---|
| 01 | 13 (ROT13) | 100 | [01-rot13.md](writeups/cryptography/01-rot13.md) |
| 02 | Easy1 (Vigenere) | 100 | [02-easy1-vigenere.md](writeups/cryptography/02-easy1-vigenere.md) |
| 03 | Mind your Ps and Qs (RSA) | 250 | [03-mind-your-ps-and-qs.md](writeups/cryptography/03-mind-your-ps-and-qs.md) |

### Web Exploitation
| # | Challenge | Points | File |
|---|---|---|---|
| 01 | Inspect HTML | 100 | [01-inspect-html.md](writeups/web-exploitation/01-inspect-html.md) |
| 02 | Get aHEAD | 100 | [02-get-ahead.md](writeups/web-exploitation/02-get-ahead.md) |
| 03 | Where are the Robots | 100 | [03-where-are-the-robots.md](writeups/web-exploitation/03-where-are-the-robots.md) |
| 04 | Logon | 100 | [04-logon.md](writeups/web-exploitation/04-logon.md) |

### Forensics
| # | Challenge | Points | File |
|---|---|---|---|
| 01 | Information | 100 | [01-information.md](writeups/forensics/01-information.md) |
| 02 | Enhance! | 100 | [02-enhance.md](writeups/forensics/02-enhance.md) |
| 03 | Redaction Gone Wrong | 100 | [03-redaction-gone-wrong.md](writeups/forensics/03-redaction-gone-wrong.md) |
| 04 | Matryoshka Doll | 300 | [04-matryoshka-doll.md](writeups/forensics/04-matryoshka-doll.md) |

### Reverse Engineering
| # | Challenge | Points | File |
|---|---|---|---|
| 01 | Safe Opener | 100 | [01-safe-opener.md](writeups/reverse-engineering/01-safe-opener.md) |
| 02 | Vault Door 1 | 100 | [02-vault-door-1.md](writeups/reverse-engineering/02-vault-door-1.md) |
| 03 | Fresh Java | 200 | [03-fresh-java.md](writeups/reverse-engineering/03-fresh-java.md) |

### Binary Exploitation
| # | Challenge | Points | File |
|---|---|---|---|
| 01 | Buffer Overflow 0 | 200 | [01-buffer-overflow-0.md](writeups/binary-exploitation/01-buffer-overflow-0.md) |
| 02 | Stonks | 200 | [02-stonks.md](writeups/binary-exploitation/02-stonks.md) |
| 03 | Flag Leak | 300 | [03-flag-leak.md](writeups/binary-exploitation/03-flag-leak.md) |

## How to use these writeups

1. Open the challenge in the [Challenge Library](https://learn.cylabacademy.org/library), read its description, and try it alone first.
2. If stuck, open the matching writeup and follow the **Recon** section before the **Solution Path**.
3. Solve it yourself end-to-end, then paste your own flag and screenshots where the writeup marks `[TODO]`. That turns the file into your personal evidence log.
4. Re-solve the challenge a week later without notes. The point value is small; the muscle memory is the real reward.

## Suggested learning order

1. General Skills 01-04 — shell confidence.
2. Web Exploitation 01-04 + Forensics 01-03 — first real vulnerability classes with zero setup.
3. Cryptography 01-03 — from classical ciphers to reading RSA parameters.
4. Reverse Engineering 01-03 — decompilation and key-check analysis.
5. Binary Exploitation 01-03 — memory corruption and format strings.

## Tooling quick reference

| Need | Tool |
|---|---|
| Shell (browser, no setup) | Platform webshell |
| Local shell | WSL / Kali / any Linux box |
| HTTP testing | `curl`, Burp Suite Community |
| Encoding / ciphers | CyberChef, `base64`, `xxd`, Python |
| File identification | `file`, `exiftool`, `strings`, `binwalk` |
| Java decompilation | jadx, CFR, recompiler.com |
| Binary analysis | gdb (+ peda/GEF), `checksec` |

## Related repositories

- [android-pentesting-cheatsheet](https://github.com/MonkeyDLuffy13/android-pentesting-cheatsheet) — Android pentesting cheatsheets + Mobile Hacking Lab writeups.
- [ios-pentesting-cheatsheet](https://github.com/MonkeyDLuffy13/ios-pentesting-cheatsheet) — iOS pentesting cheatsheets + Mobile Hacking Lab writeups.

## License

MIT — see [LICENSE](LICENSE).
