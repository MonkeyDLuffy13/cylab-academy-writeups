# CyLab Academy — Redaction Gone Wrong · Writeup

| | |
|---|---|
| Category | Forensics |
| Points | 100 |
| Tags | PDF, redaction failure, text extraction |
| Status | Solved |

> **Goal:** understand why drawing a black box over text is not redaction. PDFs separate content from presentation — the "deleted" text is still in the file, and one command recovers it.

---

## 1. Understanding the challenge
A PDF (a mock budget/bio page) with the flag covered by a black rectangle. Visually it is gone. Structurally, the rectangle is just one more drawing operator painted *over* the text glyphs, which remain fully extractable.

## 2. Tools
- `pdftotext` (poppler-utils)
- Python `pdfplumber`/`PyPDF2` as alternative
- Any PDF viewer with select/copy

## 3. Recon
```bash
file document.pdf
pdftotext document.pdf - | grep -i pico
```
The `-` sends extracted text to stdout. The flag appears even though no viewer shows it.

**[TODO]** screenshots: the black-boxed page as rendered, and the `pdftotext` output containing the flag.

## 4. Solution Path
1. Extract all text:
```bash
pdftotext document.pdf out.txt && grep -i pico out.txt
```
2. Alternatively, open the PDF in any viewer and drag-select across the black rectangle — the selectable (invisible) text highlights, and Ctrl+C copies the flag.
3. Deep-dive (optional): `qpdf --qdf --object-streams=disable document.pdf qdf.pdf` then `grep -a pico qdf.pdf` to see the raw text operator in the content stream.

Why it works: a correct redaction removes the underlying characters and re-renders the page. A drawing-over redaction only changes paint order. The text object — with font, position, and full string — stays in the content stream and in the text layer.

**[TODO]** evidence: extracted flag text alongside the rendered page.

## 5. Commands / Payloads Used
```bash
pdftotext document.pdf - | grep -i pico
qpdf --qdf --object-streams=disable in.pdf out.pdf    # raw stream inspection
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The reviewed document applied visual obfuscation (opaque rectangles) instead of true redaction (<evidence>). The covered strings remained present in the PDF text layer and were recovered with standard extraction tooling, disclosing the supposedly removed information. Remediation: use dedicated redaction features that delete content objects (Adobe Redact, `pdftk`/`qpdf` rebuilds, or re-render to raster), and verify redactions by running text extraction on every published document."

## 8. Lessons
- Black-box-over-text is the single most common redaction failure; test every "redacted" PDF you receive with `pdftotext`.
- The same logic applies to images: blurring small text is often reversible; pixelate or cut instead (and even pixelation has known attacks).
- Verification step for reports: after redacting, run extraction tools and confirm the secret is truly absent.
