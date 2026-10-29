# CyLab Academy — breadth · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium (picoMini by redpwn 2021) |
| Tags | binary diffing, dead code, radare2, patch analysis |
| Status | Solved |

> **Goal:** you get TWO builds of the same binary — an in-development version where the real-flag function "had a mistake", and the fixed one. Both are stuffed with hundreds of decoy flag strings. Diff the builds, find the one changed byte, follow it to the real flag.

---

## 1. Understanding the challenge
Two ELF files: `breadth.v1` and `breadth.v2`. Both print `Dead code? What's that? Goodbye!` when run. `strings` on either shows dozens of `picoCTF{...}` values — all decoys ("horizontal scaling" = the flag function was copy-pasted everywhere, and only one copy is real).

## 2. Tools
- `radiff2` (radare2's binary differ)
- `readelf` / `diff` (metadata comparison)
- radare2 (`aa`, `s`, `pdf`)

## 3. Recon
1. Run both: identical behavior.
2. Count the decoys:
```bash
strings breadth.v1 | grep picoCTF | head
# dozens of different flags
```
3. Diff the builds:
```bash
radiff2 breadth.v1 breadth.v2
# 0x000002d4 <build id>  =>  <different build id>
# 0x0009504b  54 => 44
# 0x0009504e  31 => 32
diff <(readelf -a breadth.v1) <(readelf -a breadth.v2)
# generated.v1.c vs generated.v2.c + new Build ID
```
Everything is identical except the build ID and **two bytes at `0x0009504b`/`0x0009504e`**.

**[TODO]** screenshot: the radiff2 output marking the changed bytes.

## 4. Solution Path
1. Examine the changed region in v2 with radare2:
```bash
r2 ./breadth.v2
[0x00001070]> aa
[0x00001070]> s 0x0009504b
[0x0009504b]> pdf
mov qword [rsp - 0x10], 0x41bc73e
mov rax, qword [rsp - 0x10]
cmp rax, 0x41bc73e          ; <- the byte 0x31 -> 0x32 lives inside this constant
je  0x95060
```
2. The description said the real-flag function had a mistake in v1 — the fix is right here. Follow the jump target:
```bash
[0x0009504b]> s 0x95060
[0x00095060]> pdf
lea rdi, str.picoCTFVnDB2LUf1VFJkdfDJtdYtFlMexPxXS6X
jmp sym.imp.puts
```
3. The real flag is the string this corrected function prints — one `lea` away from the patched comparison. Verify it against the platform.

Why it works: a two-version challenge is a controlled experiment. Build metadata (build ID, source filename) tells you where to ignore; `radiff2` collapses thousands of identical decoys into the two bytes that matter.

**[TODO]** evidence: the disassembly at 0x95060 + the accepted flag.

## 5. Commands / Payloads Used
```bash
radiff2 breadth.v1 breadth.v2
r2 -A ./breadth.v2   # then: s <addr>; pdf
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "Two builds of a binary differing in a two-byte constant were provided alongside hundreds of decoy secret strings (<evidence>). Binary diffing isolated the patch region, and following the corrected function's control flow exposed the genuine flag among the decoys. Differential analysis between software versions rapidly localizes security-relevant changes; defenders should assume released diffs leak the location and nature of security fixes."

## 8. Lessons
- `radiff2` first, Ghidra second: when two versions exist, the diff *is* the map.
- Dead code and decoy strings are noise by design — never eyeball hundreds of `strings` hits when a differ can pick the needle.
- radare2 minimal kit for this: `aa` (analyze), `s <addr>` (seek), `pdf` (disassemble function). Three commands, whole challenge.
