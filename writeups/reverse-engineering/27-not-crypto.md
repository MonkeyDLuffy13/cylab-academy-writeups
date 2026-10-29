# CyLab Academy — not crypto · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium (picoMini by redpwn 2021) |
| Tags | compiler intrinsics, avx, memcmp, dynamic analysis |
| Status | Solved |

> **Goal:** "there's crypto in here but the challenge is not crypto..." Ghidra shows a wall of vectorized AES. Do not read it. The answer is handed to `memcmp` in a register.

---

## 1. Understanding the challenge
A binary that "bargains" for a flag: you feed it input, it answers `Nope, come back later`. The decompile is hundreds of lines of AVX intrinsics — `vmovdqa`, `vpextrb`, table lookups at `DAT_001020a0` (that is the AES S-box), multiply-by-2 with `0x1b` reduction (MixColumns). The compiler auto-vectorized a full AES implementation. Hence the name: recognizing *what* it is means you know you don't have to reverse it.

## 2. Tools
- gdb with GEF (breakpoint + register read)
- Ghidra (only to find *where* the comparison happens)
- `cat /proc/<pid>/maps` or gdb `info proc mappings` (rebase addresses)

## 3. Recon
1. Run it, feed garbage, observe the failure message.
2. Skim the decompile purely for *API calls*: there is exactly one interesting one — `memcmp(controlled_buffer, expected_buffer, 0x40)`.
3. Locate the `CALL memcmp` address in Ghidra. Because the binary is PIE, note its runtime base: run under gdb (which disables randomization by default), check `info proc mappings` — the image loads at `0x0000555555554000`. Rebase Ghidra (Window -> Memory Map -> set base) so static addresses match runtime addresses.

**[TODO]** screenshot: the S-box constants in the decompile + the memcmp call site.

## 4. Solution Path
1. Break right before the comparison:
```
gef> b *0x5555555553b9
gef> r
I heard you wanted to bargain for a flag... whatcha got?
aaaa... (64 chars)
```
2. At the breakpoint, read the two `memcmp` arguments (SysV: `rdi` = arg1, `rsi` = arg2, `rdx` = length 0x40):
```
gef> printf "%s", $rdi
picoCTF{c0mp1l3r_0pt1m1z4t10n_15_pur3_w1z4rdry_but_n0_pr0bl3m?}
```
The "encrypted" buffer your input was supposed to match is the flag itself, sitting in plaintext in a register. The whole AES layer was a costume.

Why it works: every encrypted-comparison design eventually compares two buffers with `memcmp`/`strcmp`, and those arguments are materialized in registers a debugger reads freely. Vectorization changes the code, not the runtime dataflow.

**[TODO]** evidence: the breakpoint context + `printf "%s", $rdi` output.

## 5. Commands / Payloads Used
```bash
gdb -q ./not-crypto
b *0x5555555553b9        # call memcmp
r
printf "%s", $rdi
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The binary implemented AES via compiler-vectorized intrinsics to transform user input before a final comparison (<evidence>). Rather than reversing the cryptographic implementation, analysis targeted the comparison call itself, where the expected value was exposed in a general-purpose register at a breakpoint. Any local transform is transparent to runtime inspection; secrets validated in client-controlled processes must be treated as disclosed. Remediation: server-side validation for secret-bearing checks."

## 8. Lessons
- Learn to *recognize* crypto constants (S-box `63 7C 77 7B F2 6B 6F C5...`, the `0x1b` reduction) — recognition tells you what to skip.
- When you see a wall of intrinsics, grep the decompile for calls (`memcmp`, `strcmp`, `fwrite`): the interesting data flows through them.
- PIE base mismatch confusing your breakpoints? gdb's `info proc mappings` + Ghidra's Memory Map rebase aligns the two worlds in a minute.
