# CyLab Academy — Wave a Flag · Writeup

| | |
|---|---|
| Category | General Skills |
| Points | 10 |
| Tags | permissions, strings, ELF binary |
| Status | Solved |

> **Goal:** first contact with an executable file — fix its permissions, control how it runs, and fall back to `strings` when running it is not enough.

---

## 1. Understanding the challenge
You get a Linux binary called `flag` and a hint about invoking the program. A freshly downloaded file is not executable (the download strips the bit), so the challenge trains the `chmod` reflex and then shows that argument handling / static analysis can leak what the program hides.

## 2. Tools
- Linux shell
- `chmod`, `strings`, `./binary`

## 3. Recon
```bash
file flag          # ELF 64-bit executable
./flag             # Permission denied — the exec bit is missing
chmod +x flag      # add the executable bit
./flag             # runs, prints a usage/help line instead of the flag
```
**[TODO]** screenshots: permission denied before `chmod`, usage line after.

## 4. Solution Path
Two independent paths, both worth doing:

**Path A — static:** the flag is a constant baked into the binary, and printable strings are visible without executing anything:
```bash
strings flag | grep -i pico
```

**Path B — dynamic:** the help text is printed on request, and help output is usually the least-redacted output a program has:
```bash
./flag -h
```
Either way the flag is on screen. The lesson to internalize: a binary that "refuses" to cooperate still gives up its constants to `strings`, because C-style string literals live in the `.rodata` section in cleartext.

**[TODO]** evidence: `strings` output line containing the flag.

## 5. Commands Used
```bash
chmod +x flag
strings flag | grep -i pico
./flag -h
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The tested binary embedded its secret directly in the `.rodata` section, allowing full disclosure via `strings` without execution. This mirrors a real finding class: credentials, API keys, and debug strings compiled into released artifacts. Remediation: never embed secrets in binaries; store them in environment-managed configuration and audit release artifacts with `strings` in the CI pipeline."

## 8. Lessons
- Downloads lose the executable bit — `chmod +x` is always step one.
- `strings` is the fastest way to see what a binary will admit; run it before any debugger.
- Help flags (`-h`, `--help`) print information the "main path" hides — in real apps they leak versions, paths, and flags.
