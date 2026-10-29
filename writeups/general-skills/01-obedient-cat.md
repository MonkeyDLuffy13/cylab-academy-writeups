# CyLab Academy — Obedient Cat · Writeup

| | |
|---|---|
| Category | General Skills |
| Points | 5 |
| Tags | shell, files, first challenge |
| Status | Solved |

> **Goal:** break the "double-click files" habit. Download a file, look at it with the shell, and meet the flag format `picoCTF{...}`.

---

## 1. Understanding the challenge
The challenge hands you a single file named `cat` (no extension) and asks for the flag inside it. There is no trick — it is a calibration challenge that verifies you can download attachments and read them from a terminal.

## 2. Tools
- Platform webshell (or any Linux shell)
- `cat`, `file`, `ls`

## 3. Recon
```bash
ls -la                  # confirm the download landed
file cat                # ASCII text
```
`file` says the blob is plain ASCII text — no decoding, no tools, just read it.

## 4. Solution Path
```bash
cat cat
```
The file content is the flag itself. That is the entire challenge — and that is the point: before any real forensics or exploitation, you need a reflex for "what is this artifact?" answered by `file` + `cat`/`strings`, not by opening it in a GUI and hoping.

**[TODO]** screenshot: terminal showing the `file` output and the flag line.

## 5. Commands Used
```bash
file cat
cat cat
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "During triage of unknown artifacts, the standard first step is file-type identification (`file`) followed by content inspection (`cat` for text, `strings` for binaries). This workflow immediately revealed the embedded secret with no specialized tooling, demonstrating that unidentified files should never be executed or opened before their true type is confirmed."

## 8. Lessons
- `file` before everything: extensions lie, magic bytes do not.
- The webshell is enough for a large part of the library — no local VM required.
- `picoCTF{...}` is the flag wrapper everywhere on this platform; grep for it when lost.
