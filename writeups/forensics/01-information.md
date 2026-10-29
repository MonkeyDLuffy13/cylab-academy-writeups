# CyLab Academy — Information · Writeup

| | |
|---|---|
| Category | Forensics |
| Points | 100 |
| Tags | metadata, EXIF, strings |
| Status | Solved |

> **Goal:** learn that images carry hidden baggage — metadata. EXIF fields regularly leak author, GPS, software versions, and in this case, exactly the string you are looking for.

---

## 1. Understanding the challenge
An image file named `cat.jpg` with a hint that files "can be changed in a secret way". The pixels are a decoy; the interesting content is in the file structure around them.

## 2. Tools
- `exiftool`
- `strings`, `file`

## 3. Recon
```bash
file cat.jpg            # JPEG image data, EXIF standard
strings cat.jpg | grep -i pico        # cheapest possible sweep
exiftool cat.jpg        # structured metadata dump
```
`strings` may already reveal it; `exiftool` shows it cleanly in its field (e.g., an Artist/Comment tag).

**[TODO]** screenshots: exiftool output with the flag-bearing tag highlighted.

## 4. Solution Path
1. Run `exiftool` on the image.
2. Scan every field for the flag pattern — do not stop at the obvious ones (Comment, Artist, Description); flags have been hidden in Camera Model and Copyright too.
```bash
exiftool cat.jpg | grep -i pico
```
3. Submit the flag.

Why it works: JPEG (and PNG/TIFF) containers hold key-value metadata blocks written by cameras and editors. Nothing validates or strips them by default, so whatever an author (or a redaction tool) leaves there ships with the file.

**[TODO]** evidence: the grep hit.

## 5. Commands / Payloads Used
```bash
file cat.jpg
exiftool cat.jpg
strings -a cat.jpg | grep -iE 'pico|flag'
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The examined image contained unredacted metadata tags carrying the sensitive string (<evidence>). EXIF metadata is transported with every copy of a file unless actively stripped, and has repeatedly caused real-world leaks (geolocation of personnel, device identifiers, author identities). Remediation: strip metadata at ingestion/publish boundaries (`exiftool -all=`, image processing pipelines), and include metadata review in data-loss-prevention checklists."

## 8. Lessons
- Forensics triage order: `file` -> `strings` -> `exiftool`/`binwalk` -> specialized tools. The first three solve half the category.
- `strings -a` scans the whole file (not just data sections) — the `-a` matters on binaries.
- Before publishing any image in a report, strip its metadata; it is an OSINT gift.
