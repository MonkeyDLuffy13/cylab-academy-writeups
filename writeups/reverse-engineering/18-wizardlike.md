# CyLab Academy — Wizardlike · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium-Hard (picoCTF 2022) |
| Tags | maze, static analysis, strings, binary patching |
| Status | Solved |

> **Goal:** a top-down maze game gives you a few moves per run and walls block you — but the entire map lives inside the binary. Extract the map, plot the route to the flag tile, then either script the walk or patch the walls away.

---

## 1. Understanding the challenge
A terminal "RPG": you move with WASD, the screen redraws each turn. The executable contains the full level (map layout, player start, flag position) as embedded data — the display is just rendering that data. Two independent solve paths, both clean.

## 2. Tools
- `strings` / `grep` (find the map)
- Ghidra or objdump (locate the move handler)
- Python (script the move sequence)
- A hex editor or `r2 -w` (collision patch)

## 3. Recon
1. Run it: confirm WASD movement and that you hit a wall quickly.
2. Mine the data:
```bash
strings wizardlike | less
# look for repeated rows of map glyphs: walls (|, +, #), floor (.), player (@), flag tile
./wizardlike | strings   # sometimes the map is only unpacked at runtime — pipe a run and grep
```
3. If the map only exists at runtime (unpacked/decrypted in memory), dump it: run under gdb, break after the draw routine, and print the buffer — or simpler, capture many frames of stdout and stitch the visible slices into one map by hand.

**[TODO]** screenshot: the extracted map (concatenated slices or the strings hit).

## 4. Solution Path — A: script the walk
1. Plot the shortest path from the player tile to the flag tile on your extracted map (BFS by hand or in code).
2. Feed the moves to the game in one go — the game reads stdin:
```bash
python3 - <<'EOF'
import pwn
p = pwn.process('./wizardlike')
# interleave: send move, read frame, repeat — or bulk-send if the game buffers input
p.send(b'w' * 3 + b'd' * 2 + ...)   # your plotted path
print(p.recvall(timeout=5).decode(errors='replace'))
EOF
```
3. The final frame shows the flag where the flag tile was.

## Solution Path — B: patch the walls
1. In Ghidra, find the move handler: it compares the destination tile against wall constants and refuses movement. The comparison is a `cmp` + conditional branch right before the position update.
2. NOP the branch (or invert it) so every move succeeds:
```
74 xx        ->  90 90      # je -> nop  (or flip 74 -> 75)
```
3. Walk in a straight-ish line to the flag — walls no longer matter:
```bash
python3 -c "print('d'*20 + 's'*10, end='')" | ./wizardlike_patched
```

Why it works: the map is static data and the movement rules are a branch you own. Any game whose world is readable is a game whose world is editable.

**[TODO]** evidence: the winning frame with the flag / the patched bytes in Ghidra.

## 5. Commands / Payloads Used
```bash
strings wizardlike | grep -A20 -B2 '@'
# Ghidra: locate move handler, NOP the wall-check branch
python3 -c "print('w'*3+'d'*2+'s'*4, end='')" | ./patched
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The game binary embedded its full level layout and enforced movement restrictions with a client-side branch instruction (<evidence>). The map was extracted statically and either replayed as a scripted input sequence or bypassed by patching the collision check, revealing the protected content. This mirrors real game/DRM clients: any check executed where the user controls the binary is advisory only. Remediation: validate state server-side; never trust client-side enforcement for value-bearing content."

## 8. Lessons
- `strings` first, always — maze/map challenges usually ship their world as plain glyph data.
- When data is runtime-only, the fallback is frame-stitching: the renderer shows you a slice every turn; slices compose into the whole.
- Two solutions for every game RE task: play better (script) or change the rules (patch). Patching is often faster.
