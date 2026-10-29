# CyLab Academy — reverse_cipher · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium (picoCTF 2019) |
| Tags | ghidra, positional shift cipher, index parity |
| Status | Solved |

> **Goal:** a binary and its encrypted output. Ghidra reveals a positional transformation — characters shift up or down depending on index parity. Run the transform backwards and the flag reads itself.

---

## 1. Understanding the challenge
"We have recovered a binary and a text file. Can you reverse the flag." Files: `rev` (x86-64 ELF, not stripped) and `rev_this` (ASCII, no line terminator) containing `picoCTF{w1{1wq84>654f26}` — flag-shaped but scrambled after the prefix.

## 2. Tools
- Ghidra (decompiler)
- Python (inverse transform)

## 3. Recon
1. `file` both artifacts; `cat rev_this` — the first 8 characters `picoCTF{` look untouched, the tail is shifted.
2. Decompile `main` in Ghidra. The logic is short and complete:

```c
fread(flag_buf, 24, 1, flag_stream);
i = 0;
while (i < 8)                        // prefix copied verbatim
    fputc(flag_buf[i], output), i++;
j = 8;
while (j < 23) {                     // body transformed by index parity
    if ((j & 1U) == 0) c = flag_buf[j] + 5;
    else               c = flag_buf[j] - 2;
    fputc(c, output), j++;
}
fputc(flag_buf[23], output);         // closing brace verbatim
```

**[TODO]** screenshot: the decompiled transform loops.

## 4. Solution Path
Invert every operation:
- positions 0-7: unchanged
- even index in 8..22: subtract 5
- odd index in 8..22: add 2
- position 23: unchanged

```python
data = open("rev_this", "rb").read()
out = []
for i in range(8):
    out.append(chr(data[i]))
for i in range(8, 23):
    out.append(chr(data[i] - 5) if (i & 1) == 0 else chr(data[i] + 2))
out.append(chr(data[23]))
print("".join(out))
# picoCTF{r3v3rs369806a41}
```

Why it works: the cipher is a bijection indexed by position — knowledge of the transform *is* the key. The untouched prefix and suffix also give you a free sanity check: they must survive the inverse unchanged.

**[TODO]** evidence: the script output with the flag.

## 5. Commands / Payloads Used
```python
chr(data[i] - 5) if i % 2 == 0 else chr(data[i] + 2)   # i in 8..22
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The encrypted artifact was produced by a reversible positional transformation (index-parity-based character shifts) implemented in the recovered binary (<evidence>). Decompilation exposed the exact transform, and applying its inverse recovered the plaintext flag. Custom homemade ciphers without keys provide zero confidentiality once the implementing code is available — recovery is mechanical. Remediation: use vetted, keyed cryptographic primitives; never rely on obscurity of the transform."

## 8. Lessons
- Not-stripped + tiny logic = decompile and read; only reach for emulators/debuggers when the transform is genuinely complex.
- Verify your inversion on the known-plaintext regions (`picoCTF{` and `}`) before trusting the middle.
- The pattern "different rule for even/odd indices" is common in toy ciphers — check `j & 1` conditions first when skimming decompiled loops.
