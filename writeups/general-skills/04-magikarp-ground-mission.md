# CyLab Academy — Magikarp Ground Mission · Writeup

| | |
|---|---|
| Category | General Skills |
| Points | 30 |
| Tags | ssh, filesystem navigation |
| Status | Solved |

> **Goal:** a full SSH round trip — connect with a non-standard port and a password, then navigate the filesystem (`ls`, `cd`, `cat`) to collect a flag split across three directories. This is the foundation for every remote lab from here on.

---

## 1. Understanding the challenge
An SSH server hosts a Linux account. The flag is cut into three parts stored in files across three different directories, so you must move around the filesystem instead of just landing and listing your home directory. Each part tells you where the next one lives.

## 2. Tools
- `ssh` (platform webshell or local)
- `ls`, `cat`, `cd`

## 3. Recon
```bash
ssh -p <port> ctf-player@<host>
# accept host key, paste the password from the challenge page
ls -la                       # flag part 1 is right here
cat 1of3.flag.txt            # picoCTF{...; also a hint to cd /
```
**[TODO]** screenshot: the login banner and part 1.

## 4. Solution Path
1. Part 1 sits in the account's home directory and its content points to `/`.
```bash
cd /
ls
cat 2of3.flag.txt            # part 2; points to /root
```
2. `/root` is another account's home — reachable by path from here:
```bash
cd /root
ls
cat 3of3.flag.txt            # part 3
```
3. Concatenate mentally (or literally) in order: part1 + part2 + part3 = flag.

Alternative one-liner once you understand the layout (no navigation at all):
```bash
ssh -p <port> ctf-player@<host> "cat ~/1of3.flag.txt /2of3.flag.txt /root/3of3.flag.txt"
```

**[TODO]** evidence: the three `cat` outputs joined into the full flag.

## 5. Commands Used
```bash
ssh -p <port> ctf-player@<host>
cat 1of3.flag.txt
cd / && cat 2of3.flag.txt
cd /root && cat 3of3.flag.txt
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The reviewed SSH-facing account was able to read files owned by other system users, including material under `/root`, indicating over-broad file permissions or group memberships. On a real host this converts a low-privilege shell compromise into full disclosure of other services' data. Remediation: principle of least privilege on home directories and service data, and SSH hardening (key-only auth, restricted shells)."

## 8. Lessons
- Non-standard port + password: `ssh -p` is the only extra step — memorize it.
- Hints inside challenge text are navigation instructions; read file contents fully before moving on.
- Non-interactive command execution over SSH (`ssh host "cmd"`) is how you will automate later labs.
