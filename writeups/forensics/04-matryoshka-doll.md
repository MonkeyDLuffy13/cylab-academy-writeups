# CyLab Academy — Matryoshka Doll · Writeup

| | |
|---|---|
| Category | Forensics |
| Points | 300 |
| Tags | binwalk, data carving, nested files |
| Status | Solved |

> **Goal:** carve files out of files. Images are containers: appended data and embedded files survive inside them, and the doll opens layer by layer with the same two commands.

---

## 1. Understanding the challenge
`doll.jpg` — a picture of matryoshka dolls (on theme). The hint says files inside files. Somewhere after the JPEG end-of-image marker (`FF D9`) sits another file, which itself contains another, until one layer holds the flag.

## 2. Tools
- `binwalk` (signature scan + extraction)
- `foremost` or `dd` as fallback
- `file`, `xxd`

## 3. Recon
```bash
file doll.jpg                       # JPEG
binwalk doll.jpg
# DECIMAL  HEXADECIMAL  DESCRIPTION
# ...
# <offset>  0x...        PNG image, 800 x 1000
xxd doll.jpg | grep -m1 'ffd9'       # end-of-image marker precedes the payload
```
A JPEG's trailer is `FF D9`; anything after it is not "part of the picture" — it is cargo.

**[TODO]** screenshot: binwalk output showing the embedded signature.

## 4. Solution Path
1. Extract automatically:
```bash
binwalk -e doll.jpg            # carves into ./_doll.jpg.extracted/
cd _doll.jpg.extracted && file *
```
2. You now have the next doll (an PNG image containing another image). Repeat the same loop:
```bash
binwalk -e <extracted-file>
```
3. Each layer: `file` it, `binwalk` it, extract, repeat — the flag image appears after 2-3 rounds (`base_images/<n>.txt` style payload at the end).
4. Manual fallback (understand what binwalk automates):
```bash
python3 - <<'EOF'
data = open('doll.jpg','rb').read()
i = data.find(b'\x89PNG')
open('layer2.png','wb').write(data[i:])
EOF
```
or `foremost -i doll.jpg -o out/`.

**[TODO]** evidence: each extraction round's `file` output, and the final flag image.

## 5. Commands / Payloads Used
```bash
binwalk -e <file>          # signature scan + extraction loop
file *                     # identify what fell out
foremost -i in -o out      # carving fallback
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "Static analysis of the supplied image identified data appended beyond the format's end-of-stream marker (<evidence>), allowing complete recovery of the embedded files via signature-based carving. Container formats tolerate trailing garbage, which makes steganography-by-appending trivial to perform and, in incident response, a common malware-payload carrier (image spam, polyglot files). Remediation for defenders: normalize/re-encode uploaded media, reject files with trailing data, and alert on images whose byte size far exceeds their rendered dimensions."

## 8. Lessons
- The loop `file -> binwalk -> binwalk -e -> repeat` solves the entire "nested files" family.
- Know the format sentinels you grep for: JPEG `FF D9`, PNG `\x89PNG`, ZIP `PK\x03\x04`.
- `binwalk -e` output lands in `_<name>.extracted/` — check `file *` there before diving deeper; sometimes the answer is already carved.
