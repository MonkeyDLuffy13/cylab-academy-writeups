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
| # | Challenge | Difficulty | File |
|---|---|---|---|
| 01 | Inspect HTML | Easy | [01-inspect-html.md](writeups/web-exploitation/01-inspect-html.md) |
| 02 | Get aHEAD | Easy | [02-get-ahead.md](writeups/web-exploitation/02-get-ahead.md) |
| 03 | Where are the Robots | Easy | [03-where-are-the-robots.md](writeups/web-exploitation/03-where-are-the-robots.md) |
| 04 | Logon | Easy | [04-logon.md](writeups/web-exploitation/04-logon.md) |
| 05 | Search source | Medium | [05-search-source.md](writeups/web-exploitation/05-search-source.md) |
| 06 | Roboto Sans | Medium | [06-roboto-sans.md](writeups/web-exploitation/06-roboto-sans.md) |
| 07 | Power Cookie | Medium | [07-power-cookie.md](writeups/web-exploitation/07-power-cookie.md) |
| 08 | Forbidden Paths | Medium | [08-forbidden-paths.md](writeups/web-exploitation/08-forbidden-paths.md) |
| 09 | SQLiLite | Medium | [09-sqlilite.md](writeups/web-exploitation/09-sqlilite.md) |
| 10 | SQL Direct | Medium | [10-sql-direct.md](writeups/web-exploitation/10-sql-direct.md) |
| 11 | MatchTheRegex | Medium | [11-match-the-regex.md](writeups/web-exploitation/11-match-the-regex.md) |
| 12 | SOAP (XXE) | Medium | [12-soap-xxe.md](writeups/web-exploitation/12-soap-xxe.md) |
| 13 | More SQLi | Medium | [13-more-sqli.md](writeups/web-exploitation/13-more-sqli.md) |
| 14 | picobrowser | Medium | [14-picobrowser.md](writeups/web-exploitation/14-picobrowser.md) |
| 15 | Irish-Name-Repo 1 | Medium | [15-irish-name-repo-1.md](writeups/web-exploitation/15-irish-name-repo-1.md) |
| 16 | Secrets | Medium | [16-secrets.md](writeups/web-exploitation/16-secrets.md) |
| 17 | It is my Birthday | Medium | [17-it-is-my-birthday.md](writeups/web-exploitation/17-it-is-my-birthday.md) |
| 18 | findme (OAuth) | Medium | [18-findme.md](writeups/web-exploitation/18-findme.md) |
| 19 | JAuth (JWT alg:none) | Medium | [19-jauth.md](writeups/web-exploitation/19-jauth.md) |
| 20 | Most Cookies (Flask session) | Medium | [20-most-cookies.md](writeups/web-exploitation/20-most-cookies.md) |
| 21 | Super Serial (Java deserialization) | Medium | [21-super-serial.md](writeups/web-exploitation/21-super-serial.md) |
| 22 | Web Gauntlet (SQLi filter evasion) | Medium | [22-web-gauntlet.md](writeups/web-exploitation/22-web-gauntlet.md) |
| 23 | caas (command injection) | Medium | [23-caas.md](writeups/web-exploitation/23-caas.md) |
| 24 | Client-side-again (obfuscated JS) | Medium | [24-client-side-again.md](writeups/web-exploitation/24-client-side-again.md) |
| 25 | JaWT Scratchpad (secret cracking) | Medium | [25-jawt-scratchpad.md](writeups/web-exploitation/25-jawt-scratchpad.md) |
| 26 | Java Code Analysis!?! (JWT forge) | Medium | [26-java-code-analysis.md](writeups/web-exploitation/26-java-code-analysis.md) |
| 27 | More Cookies (CBC bit flip) | Medium | [27-more-cookies.md](writeups/web-exploitation/27-more-cookies.md) |
| 28 | No Sql Injection (Mongo regex oracle) | Medium | [28-no-sql-injection.md](writeups/web-exploitation/28-no-sql-injection.md) |
| 29 | SSTI2 (Jinja2 filter evasion) | Medium | [29-ssti2.md](writeups/web-exploitation/29-ssti2.md) |
| 30 | 3v@l (Python eval jail) | Medium | [30-3val.md](writeups/web-exploitation/30-3val.md) |
| 31 | WebSockFish (WS frame tampering) | Medium | [31-websockfish.md](writeups/web-exploitation/31-websockfish.md) |
| 32 | Trickster (upload magic-byte bypass) | Medium | [32-trickster.md](writeups/web-exploitation/32-trickster.md) |
| 33 | Java Script Kiddie (client-side brute force) | Medium | [33-javascript-kiddie.md](writeups/web-exploitation/33-javascript-kiddie.md) |
| 34 | X marks the spot (XPath injection oracle) | Medium | [34-x-marks-the-spot.md](writeups/web-exploitation/34-x-marks-the-spot.md) |
| 35 | login (client-side creds, picoMini redpwn) | Easy | [35-login.md](writeups/web-exploitation/35-login.md) |
| 36 | Irish-Name-Repo 2 (filtered field SQLi) | Medium | [36-irish-name-repo-2.md](writeups/web-exploitation/36-irish-name-repo-2.md) |
| 37 | Irish-Name-Repo 3 (ROT13 evasion) | Medium | [37-irish-name-repo-3.md](writeups/web-exploitation/37-irish-name-repo-3.md) |
| 38 | Web Gauntlet 2 (SQLite glob tautology) | Medium | [38-web-gauntlet-2.md](writeups/web-exploitation/38-web-gauntlet-2.md) |
| 39 | Web Gauntlet 3 (payload golf, 25 chars) | Hard | [39-web-gauntlet-3.md](writeups/web-exploitation/39-web-gauntlet-3.md) |
| 40 | Java Script Kiddie 2 (known-plaintext brute) | Hard | [40-javascript-kiddie-2.md](writeups/web-exploitation/40-javascript-kiddie-2.md) |
| 41 | msfroggenerator2 (proxy + bot + file:// chain) | Hard | [41-msfroggenerator2.md](writeups/web-exploitation/41-msfroggenerator2.md) |
| 42 | Old Sessions (session hijacking, picoCTF 2026) | Easy | [42-old-sessions.md](writeups/web-exploitation/42-old-sessions.md) |
| 43 | Crack the Gate 1 (ROT13 note, header backdoor) | Easy | [43-crack-the-gate-1.md](writeups/web-exploitation/43-crack-the-gate-1.md) |
| 44 | SSTI1 (Jinja2 template injection) | Easy | [44-ssti1.md](writeups/web-exploitation/44-ssti1.md) |
| 45 | n0s4n1ty 1 (PHP webshell upload + sudo) | Easy | [45-n0s4n1ty-1.md](writeups/web-exploitation/45-n0s4n1ty-1.md) |
| 46 | head-dump (heapsnapshot secret disclosure) | Easy | [46-head-dump.md](writeups/web-exploitation/46-head-dump.md) |
| 47 | Cookie Monster Secret Recipe (Base64 cookie) | Easy | [47-cookie-monster-secret-recipe.md](writeups/web-exploitation/47-cookie-monster-secret-recipe.md) |
| 48 | WebDecode (hidden attribute, Base64) | Easy | [48-webdecode.md](writeups/web-exploitation/48-webdecode.md) |
| 49 | Unminify (beautify minified source) | Easy | [49-unminify.md](writeups/web-exploitation/49-unminify.md) |
| 50 | IntroToBurp (proxy, missing-param OTP bypass) | Easy | [50-introtoburp.md](writeups/web-exploitation/50-introtoburp.md) |
| 51 | Bookmarklet (javascript: URL, hex escapes) | Easy | [51-bookmarklet.md](writeups/web-exploitation/51-bookmarklet.md) |
| 52 | Local Authority (client-side auth logic) | Easy | [52-local-authority.md](writeups/web-exploitation/52-local-authority.md) |
| 53 | Includes (flag split across css/js includes) | Easy | [53-includes.md](writeups/web-exploitation/53-includes.md) |
| 54 | Cookies (cookie index enumeration) | Easy | [54-cookies.md](writeups/web-exploitation/54-cookies.md) |
| 55 | Scavenger Hunt (5-file hunt incl. .htaccess) | Easy | [55-scavenger-hunt.md](writeups/web-exploitation/55-scavenger-hunt.md) |
| 56 | dont-use-client-side (substring fragments) | Easy | [56-dont-use-client-side.md](writeups/web-exploitation/56-dont-use-client-side.md) |
| 57 | Insp3ct0r (view-source trilogy) | Easy | [57-insp3ct0r.md](writeups/web-exploitation/57-insp3ct0r.md) |

Writeups 05-18 were prioritized from the current CyLab library Medium tier (highest solve counts first). Writeups 42-57 cover the library's Easy Web Exploitation tier; the Easy entries Inspect HTML, GET aHEAD, where are the robots, and logon map to writeups 01-04 above.

### Forensics
| # | Challenge | Points | File |
|---|---|---|---|
| 01 | Information | 100 | [01-information.md](writeups/forensics/01-information.md) |
| 02 | Enhance! | 100 | [02-enhance.md](writeups/forensics/02-enhance.md) |
| 03 | Redaction Gone Wrong | 100 | [03-redaction-gone-wrong.md](writeups/forensics/03-redaction-gone-wrong.md) |
| 04 | Matryoshka Doll | 300 | [04-matryoshka-doll.md](writeups/forensics/04-matryoshka-doll.md) |

### Reverse Engineering
| # | Challenge | Difficulty | File |
|---|---|---|---|
| 01 | Safe Opener (Base64) | Easy | [01-safe-opener.md](writeups/reverse-engineering/01-safe-opener.md) |
| 02 | Vault Door 1 (index map) | Easy | [02-vault-door-1.md](writeups/reverse-engineering/02-vault-door-1.md) |
| 03 | Fresh Java (decompilation) | Medium | [03-fresh-java.md](writeups/reverse-engineering/03-fresh-java.md) |
| 04 | Vault Door 7 (endianness) | Hard | [04-vault-door-7.md](writeups/reverse-engineering/04-vault-door-7.md) |
| 05 | Vault Door 8 (bitwise brute force) | Hard | [05-vault-door-8.md](writeups/reverse-engineering/05-vault-door-8.md) |
| 06 | Droids 0 (logcat) | Hard | [06-droids0.md](writeups/reverse-engineering/06-droids0.md) |
| 07 | Droids 1 (jadx static) | Hard | [07-droids1.md](writeups/reverse-engineering/07-droids1.md) |
| 08 | Droids 2 (native JNI library) | Hard | [08-droids2.md](writeups/reverse-engineering/08-droids2.md) |
| 09 | Droids 3 (smali patching) | Hard | [09-droids3.md](writeups/reverse-engineering/09-droids3.md) |
| 10 | Droids 4 (AES reversing / Frida) | Hard | [10-droids4.md](writeups/reverse-engineering/10-droids4.md) |
| 11 | asm 2/3/4 (x86 series guide) | Hard | [11-asm-series.md](writeups/reverse-engineering/11-asm-series.md) |
| 12 | ARMssembly 2/3/4 (aarch64 guide) | Hard | [12-armssembly-series.md](writeups/reverse-engineering/12-armssembly-series.md) |
| 13 | Need For Speed (time gate bypass) | Hard | [13-need-for-speed.md](writeups/reverse-engineering/13-need-for-speed.md) |
| 14 | Checkpass (angr symbolic execution) | Hard | [14-checkpass.md](writeups/reverse-engineering/14-checkpass.md) |
| 15 | Keygenme (keygen / z3) | Hard | [15-keygenme.md](writeups/reverse-engineering/15-keygenme.md) |
| 16 | Virtual Machine 1 (custom VM) | Hard | [16-virtual-machine-1.md](writeups/reverse-engineering/16-virtual-machine-1.md) |
| 17 | Powershelly (encoded PS payload) | Hard | [17-powershelly.md](writeups/reverse-engineering/17-powershelly.md) |
| 18 | Wizardlike (maze map extraction) | Medium | [18-wizardlike.md](writeups/reverse-engineering/18-wizardlike.md) |
| 19 | Forky (fork count arithmetic) | Hard | [19-forky.md](writeups/reverse-engineering/19-forky.md) |
| 20 | gogo (Go binary reversing) | Hard | [20-gogo.md](writeups/reverse-engineering/20-gogo.md) |
| 21 | Let's get dynamic (ltrace / LD_PRELOAD) | Medium | [21-lets-get-dynamic.md](writeups/reverse-engineering/21-lets-get-dynamic.md) |
| 22 | riscy business (RISC-V reading) | Medium | [22-riscy-business.md](writeups/reverse-engineering/22-riscy-business.md) |
| 23 | B1ll_Gat35 (Windows x64dbg) | Hard | [23-b1ll-gat35.md](writeups/reverse-engineering/23-b1ll-gat35.md) |
| 24 | reverse_cipher (positional shifts) | Medium | [24-reverse-cipher.md](writeups/reverse-engineering/24-reverse-cipher.md) |
| 25 | Easy as GDB (character oracle) | Medium | [25-easy-as-gdb.md](writeups/reverse-engineering/25-easy-as-gdb.md) |
| 26 | OTP Implementation (gdb key recovery + XOR) | Hard | [26-otp-implementation.md](writeups/reverse-engineering/26-otp-implementation.md) |
| 27 | not crypto (memcmp register read) | Medium | [27-not-crypto.md](writeups/reverse-engineering/27-not-crypto.md) |
| 28 | breadth (binary diffing, redpwn) | Medium | [28-breadth.md](writeups/reverse-engineering/28-breadth.md) |
| 29 | No way out (Unity memory teleport) | Medium | [29-no-way-out.md](writeups/reverse-engineering/29-no-way-out.md) |

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
2. Web Exploitation 01-04 + 57, 53 — first real vulnerability classes with zero setup: view-source and its variants.
3. Web Exploitation 48-56 — the rest of the Easy tier: encoding, cookies, client-side auth, first Burp flight.
4. Web Exploitation 05-18 — the medium tier: SQLi, XXE, header and cookie manipulation, OAuth flows.
5. Cryptography 01-03 — from classical ciphers to reading RSA parameters.
6. Web Exploitation 19-28 — the JWT ladder, session forging, deserialization and injection sinks.
7. Reverse Engineering 01-05, 13, 17 — decompilation, the vault-door series, gates and encoded payloads.
8. Reverse Engineering 06-10 (droids) — the compressed Android pentest arc; pairs with the android-pentesting-cheatsheet repo.
9. Reverse Engineering 11, 12, 14-16, 22 — assembly reading (x86, ARM, RISC-V), symbolic execution, and devirtualization.
10. Web Exploitation 29-34, 42-47 — sandbox escapes, XPath injection, protocol-level tampering, and the modern 2025-2026 Easy set (SSTI, uploads, heap dumps, sessions).
11. Reverse Engineering 18-21, 23-29 — game logic, process arithmetic, Go binaries, debugger oracles, and dynamic analysis.
12. Web Exploitation 35-41 — the classic ladder: client-side secrets, filtered SQLi series, and a full proxy-bot exploitation chain.
13. Binary Exploitation 01-03 — memory corruption and format strings.

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
