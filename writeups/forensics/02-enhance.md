# CyLab Academy — Enhance! · Writeup

| | |
|---|---|
| Category | Forensics |
| Points | 100 |
| Tags | SVG, layers, XML |
| Status | Solved |

> **Goal:** recognize vector graphics as readable XML. The "enhance" trope from crime shows is real here — the flag is hidden in an invisible layer, and SVG layers are plain text you can read.

---

## 1. Understanding the challenge
You get an SVG file showing a flag graphic with a barely visible green "flag" and a hidden text element. Raster enhancement tools (GIMP filters) are a trap — SVG is not pixels, it is a scene description.

## 2. Tools
- Text editor / `cat`
- `grep`
- Optional: Inkscape or a browser (to render layers visually)

## 3. Recon
```bash
file flag.svg           # SVG Scalable Vector Graphics image
head -40 flag.svg       # plain XML
grep -in 'tspan\|text\|pico' flag.svg
```
The XML contains multiple `<g>` layer groups; one of them is (nearly) invisible — typically `fill` with a tiny alpha, an opacity of 0, or coordinates off-view.

**[TODO]** screenshot: the grep hit showing the hidden `<text>` element.

## 4. Solution Path
1. Locate the hidden text node:
```bash
grep -o '<tspan[^<]*</tspan>' flag.svg
```
2. Read its content — that is the flag. No rendering needed.
3. To *see* it for a screenshot: open the SVG in a browser, then in DevTools set the hidden layer's `opacity` to 1 — or open it in Inkscape and toggle layer visibility (Layer dialog).

Why it works: SVG stores every drawable element (including "invisible" ones) as text in the file. Visibility is just presentation attributes; the data is fully present regardless.

**[TODO]** evidence: the text content + a rendered screenshot with the layer made visible.

## 5. Commands / Payloads Used
```bash
grep -in 'opacity\|alpha\|tspan' flag.svg
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The reviewed SVG document contained content-bearing elements outside the visible layer, retrievable by reading the file as XML (<evidence>). Vector formats preserve all authored objects — including those hidden via opacity or clipping — so visual inspection alone cannot certify what a file discloses. Remediation: sanitize SVGs before publication (flatten/remove hidden layers, or rasterize), and treat SVG as an XML document (which also carries its own XSS risk when user-uploaded)."

## 8. Lessons
- "Enhance" fails on vector graphics by design — go read the XML instead.
- When a forensics challenge gives you a *format*, ask what plaintext structure that format embeds (SVG=XML, DOCX/PPTX=XLS-zip of XML, PDF=object graph).
- User-uploaded SVG is also a stored-XSS vector; note it for web reports.
