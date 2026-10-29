# CyLab Academy — Virtual Machine 1 · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Hard (picoCTF 2023) |
| Tags | custom VM, bytecode, devirtualization |
| Status | Solved |

> **Goal:** the binary embeds its own virtual machine: a bytecode interpreter with custom opcodes, and the "program" you must understand is data. The skill — building a disassembler for someone else's CPU — is the devirtualization workflow used against real obfuscated malware and license VMs.

---

## 1. Understanding the challenge
`vm1` runs a bytecode program that performs the flag/password check. Reading the interpreter's machine code tells you the VM's instruction set; disassembling the bytecode blob then reveals what the embedded program actually does.

## 2. Tools
- Ghidra (decompile the interpreter loop)
- Python (build the disassembler + emulate)
- Hex viewer for the bytecode section

## 3. Recon
1. Load in Ghidra; find the interpreter loop — the tell is a `switch` on a fetched byte:
```c
while (running) {
    op = program[pc++];
    switch (op) {
        case 0x??: /* push  */ ...
        case 0x??: /* add   */ ...
        case 0x??: /* jne   */ ...
        case 0x??: /* read input char */ ...
    }
}
```
2. Enumerate every case: note opcode value -> mnemonic -> operand width (does it consume 1, 2, 4 more bytes?).
3. Locate the program bytes (the blob the interpreter walks) and where input flows in.

**[TODO]** screenshot: the switch table with case labels.

## 4. Solution Path
1. **Write the disassembler** — a dict and a walk:
```python
OPS = { 0x01:('push',2), 0x02:('add',0), 0x03:('xor',0),
        0x04:('jz',2),  0x05:('cmp',0),  0x06:('readc',0),
        0x07:('putc',0), 0xFF:('halt',0) }        # transcribe from Ghidra!

prog = open('program.bin','rb').read()            # extract the blob
pc = 0
while pc < len(prog):
    op = prog[pc]
    name, arglen = OPS[op]
    args = prog[pc+1:pc+1+arglen]
    print(f"{pc:04x}: {name} {args.hex()}")
    pc += 1 + arglen
```
2. **Read the recovered listing**: it is now a small flag check in plain steps — typically "read char, xor/mix, compare against table entry, branch". Extract the expected table and invert (per-character, as always).
3. Alternative shortcut once opcodes are known: **emulate** the VM in Python on candidate inputs until the success path hits — useful when the program's math is messier than its structure.

Why it works: a custom VM is obfuscation of *structure*, not of *information*. The interpreter must contain a complete, unambiguous description of every operation — recover the map and the "encrypted" program reads like source.

**[TODO]** evidence: opcode table + disassembled program listing + the recovered input/flag.

## 5. Commands / Payloads Used
```python
OPS = {...}; pc-walk with per-op operand widths       # the disassembler
emulate(vm, candidate_input)                          # optional emulator shortcut
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The analyzed binary embeds a custom bytecode interpreter to obscure its validation logic (<evidence>). Decompilation of the dispatch loop recovered the instruction set, and a purpose-built disassembler restored the embedded program to readable form, defeating the control-flow obfuscation. Devirtualization remains a tractable, systematic process; custom-VM protection raises analysis cost but cannot conceal behavior from a determined static review."

## 8. Lessons
- The tell of a VM: one hot loop + a `switch` on program bytes. Find it, map it, then the challenge becomes "write a toy disassembler".
- Disassemble before emulating: the listing usually shows the whole check in 20 lines, no emulation needed.
- This workflow transfers directly to VMProtect-style packers and VM-based malware families — Virtual Machine 1 is the kid version of a real devirtualization engagement.
