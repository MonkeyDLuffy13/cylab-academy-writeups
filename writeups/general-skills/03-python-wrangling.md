# CyLab Academy — Python Wrangling · Writeup

| | |
|---|---|
| Category | General Skills |
| Points | 10 |
| Tags | python, argparse, decryption |
| Status | Solved |

> **Goal:** run someone else's script correctly — read its interface (`--help`), supply the right arguments, and use a password file for symmetric decryption. This is the daily loop of every CTF helper script.

---

## 1. Understanding the challenge
Three files: `ende.py` (encrypt/decrypt helper using `cryptography.Fernet`), `pw.txt` (the Fernet key material), and `flag.txt.en` (the encrypted flag). The task is to decrypt `flag.txt.en`. All the "hard" work is done; the skill being trained is reading a tool's usage and driving it.

## 2. Tools
- Python 3 (webshell has it)
- `cat`, script `-h` interface

## 3. Recon
```bash
python3 ende.py -h
# usage: ende.py (-e/-d) [file]
#   -d : decrypt, -e : encrypt
cat pw.txt            # the key/password for Fernet
head flag.txt.en      # base64-looking blob -> symmetric ciphertext
```
**[TODO]** screenshot: the `-h` output.

## 4. Solution Path
1. Decrypt mode needs the file and (when stdin is not a TTY) the password:
```bash
python3 ende.py -d flag.txt.en
# prompts: "Please enter the password:"  -> paste the value from pw.txt
```
2. Non-interactive variant (what you will use inside scripted solve pipelines):
```bash
cat pw.txt | python3 ende.py -d flag.txt.en
```
The flag prints to stdout.

Why it works: Fernet is authenticated symmetric encryption; the script holds the exact key in `pw.txt`. A "protected" file whose key sits next to it is confidentiality theater — a pattern that shows up constantly in misconfigured deployments.

**[TODO]** evidence: terminal showing the decrypted flag.

## 5. Commands Used
```bash
python3 ende.py -h
cat pw.txt | python3 ende.py -d flag.txt.en
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The evaluated configuration stored encryption key material alongside the ciphertext it protects (`pw.txt` next to `flag.txt.en`). Any party with file-system read access obtains both and defeats the scheme entirely, as demonstrated by direct decryption (<evidence>). Remediation: externalize keys to a secret manager with access control, and treat key-next-to-data storage as unencrypted."

## 8. Lessons
- Always check a tool's `-h` before guessing its interface — argparse is self-documenting.
- Pipe input (`cat key | script`) to make interactive tools scriptable.
- Encryption with the key stored next to the data equals no encryption; note this pattern for future audit reports.
