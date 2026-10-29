# CyLab Academy — Let's get dynamic · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium-Hard (picoCTF 2022) |
| Tags | dynamic linking, ltrace, library interposition, LD_PRELOAD |
| Status | Solved |

> **Goal:** the binary leans on a dynamically linked library for its critical comparison. Watch the library calls at runtime — or replace the library — and the secret falls out without any static reversing of the main binary.

---

## 1. Understanding the challenge
An ELF that validates your input, with the interesting logic delegated to library function calls (`strcmp`-family, custom `.so` functions, or a chain of libc calls carrying your input in their arguments). Dynamic analysis tools make those calls — and their arguments — fully visible.

## 2. Tools
- `ldd` (map the dependencies)
- `ltrace` (library call tracing) / `strace` (syscalls)
- gdb with breakpoints on PLT entries
- `LD_PRELOAD` shim (when you want to *change* behavior, not just watch)

## 3. Recon
```bash
ldd ./lets-get-dynamic           # which libraries carry the logic?
strings ./lets-get-dynamic | grep -iE 'flag|correct|wrong'
ltrace ./lets-get-dynamic        # first pass: see every library call with arguments
```
Feed a test input and watch which call receives it — that call is the comparison you need to understand or intercept.

**[TODO]** screenshot: the ltrace output showing your input inside a library call.

## 4. Solution Path
1. **Read the answer off the trace.** Most runs end here: the comparison function's argument *is* the expected secret.
```
strcmp("your_test_input", "the_real_expected_value") = ...
```
2. If the call is length/character-by-character (`strncpy`, `memcmp` on chunks), reconstruct the full value from the argument bytes across the trace:
```bash
ltrace ./lets-get-dynamic 2>&1 | grep -E 'memcmp|strcmp|strncmp'
```
3. **Interpose when watching is not enough.** If arguments are obfuscated before comparison, shim the function to log post-processing state:
```c
// shim.c
#define _GNU_SOURCE
#include <dlfcn.h>
#include <stdio.h>
int strcmp(const char *a, const char *b) {
    static int (*real)(const char*, const char*);
    if (!real) real = dlsym(RTLD_NEXT, "strcmp");
    fprintf(stderr, "[strcmp] '%s' vs '%s'\n", a, b);
    return real(a, b);
}
```
```bash
gcc -shared -fPIC -o shim.c.so shim.c -ldl
LD_PRELOAD=$PWD/shim.c.so ./lets-get-dynamic
```
4. **gdb alternative** — break at the PLT stub and inspect registers (SysV: `rdi`=arg1, `rsi`=arg2):
```bash
gdb ./lets-get-dynamic
b strcmp
run
x/s $rdi
x/s $rsi
```

Why it works: dynamic linking resolves calls to fixed library entry points at load time — every argument those functions receive is observable and replaceable from outside the binary. The program cannot distinguish a real `strcmp` from a logging one.

**[TODO]** evidence: the traced/shimmed call revealing the expected value + flag output.

## 5. Commands / Payloads Used
```bash
ltrace ./lets-get-dynamic
LD_PRELOAD=$PWD/shim.c.so ./lets-get-dynamic
gdb -ex 'b strcmp' -ex run -ex 'x/s $rdi' -ex 'x/s $rsi' ./lets-get-dynamic
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "Secret material was transmitted as an argument to a dynamically linked library function during input validation (<evidence>). Runtime tracing and symbol interposition exposed the argument without any static analysis of the application logic. Dynamic linking makes all inter-module calls observable and replaceable by the operator of the environment; therefore client-side secrets and local license checks are unenforceable against a hostile local user. Remediation: server-side validation for any secret-bearing decision."

## 8. Lessons
- Always `ltrace` before decompiling — the flag is so often sitting in a `strcmp` argument.
- `LD_PRELOAD` is watching *and* rewriting: same shim can force comparisons to succeed (`return 0`), not just log them.
- The gdb PLT-breakpoint route works when ltrace is missing or the call is in a custom `.so` (adjust the breakpoint to the custom symbol name from `nm -D`).
