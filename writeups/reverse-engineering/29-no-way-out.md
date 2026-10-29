# CyLab Academy — No way out · Writeup

| | |
|---|---|
| Category | Reverse Engineering |
| Difficulty | Medium (picoCTF 2023) |
| Tags | unity, game hacking, cheat engine, memory editing |
| Status | Solved |

> **Goal:** a Unity 3D escape room. The flag is mounted on the wall where you can *see* it but never *reach* it — invisible walls keep you locked in. Edit your player coordinates in memory and teleport to it.

---

## 1. Understanding the challenge
A downloadable Windows Unity game. You spawn in a room, climb a ladder, and spot the flag rendered in the world — behind a barrier the collision system will not let you cross. Nothing about the *code* needs reversing: the attack surface is the game's own state (player position) living in process memory.

## 2. Tools
- **Cheat Engine** (memory scanner / editor)
- The game itself, running

## 3. Recon
1. Launch the game, walk around, confirm the flag's visual location and the invisible wall.
2. Attach Cheat Engine to the game process (`File -> Open Process`).
3. Plan the scan: Unity stores player position as three 4-byte floats (`x`, `y`, `z`) adjacent in memory. Your lever is the **y-axis** (altitude), because vertical movement gives clean "high vs low" value pairs for the scanner.

**[TODO]** screenshot: the in-game view of the unreachable flag.

## 4. Solution Path
1. **Find the y-coordinate.** Note your current altitude, scan Cheat Engine for a float value in range. Climb the ladder (value increases) -> `Next scan` (increased value). Descend -> `Next scan` (decreased). A few altitude cycles isolate the single address holding your height.
2. **Derive x and z.** The three coordinate floats sit 4 bytes apart. Add the y address to your Cheat Engine list, then manually add entries at `addr+4` and `addr+8` (or `-4`) and watch which ones move when you strafe or walk forward.
3. **Teleport.** With the flag's world position estimated (walk as close as the wall allows and read your coordinates, or judge the offset from the map), write large values into x/z (and y if needed) until the camera lands beside the flag.
4. Read the screen — the flag texture is now close enough to read, completing the "escape".

Why it works: the wall is only a *rule* — collision is enforced by the same CPU that holds your position floats. Client-authoritative movement is the classic game-hacking flaw; the server never gets a vote.

**[TODO]** evidence: the Cheat Engine address list + the flag close-up.

## 5. Commands / Payloads Used
```
Cheat Engine: float scan -> increase/decrease refinement -> write x/y/z
```

## 6. Flag
```
**[TODO]** picoCTF{...}
```

## 7. Write It Up (report-ready paragraph)
> "The game enforced its level boundaries through client-side collision checks over in-memory player coordinates (<evidence>). A memory scanner isolated the position floats by correlation with vertical movement, and direct writes teleported the player through the barrier to the restricted content. Any anti-cheat or DRM logic that depends solely on client memory integrity is bypassable with commodity tooling; authoritative state must be validated server-side."

## 8. Lessons
- Value-correlation scanning (move up -> increased, move down -> decreased) is the universal recipe for finding game state without symbols.
- Coordinate triples are contiguous (`x, y, z` at 4-byte strides) — one found address usually gives you the other two for free.
- Liveoverflow's "hacking multiplayer games" videos teach exactly this workflow; this challenge is its safest practice ground.
