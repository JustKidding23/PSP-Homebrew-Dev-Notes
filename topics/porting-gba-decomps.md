# Porting GBA decompilations (pret) to the PSP

Traps found while bringing pret's decomps over to the PSP as native code (not emulation).

### GCC reorders globals that GBA code expects side by side
- **Seen as:** Pokémon FireRed's sound stopped for good a few seconds into the intro, and Oak's
  speech then waited forever on a Pokémon cry.
- **Cause:** FireRed declares `gSaveBlock2` and then `gSaveBlock2_DMA` (padding), clears both with
  one `CpuFill16(&gSaveBlock2, sizeof block + sizeof padding)`, and moves the block's pointer up
  to 127 bytes into the padding at random. The GBA linker keeps source order. GCC on the PSP put
  the padding first, so the clear ran past the block into the next variable (`gSoundInit`).
- **Workaround:** build with `-fno-toplevel-reorder` (FireRed's own modern GBA build uses it too),
  then check the order with `psp-nm -n`. pret's Emerald wraps block and padding in one struct,
  which is order-proof.
- **Confirmed:** PPSSPP 1.20 and pspdev GCC 15.2.
- **Found in:** Pokémon FireRed PSP, 2026-10-17.

### The I/O registers don't all start at 0
- **Seen as:** an affine background (FireRed's Professor Oak) shows nothing.
- **Cause:** at power-on the GBA has BG2PA, BG2PD, BG3PA and BG3PD at 0x100 (a scale of 1). A port
  that zeroes its register array draws every pixel of the layer from its top-left corner, which is
  usually clear. Games rely on the power-on values and never set them.
- **Workaround:** set those four to 0x100 at startup and on every register reset, like mGBA's
  `GBAIOInit` (`src/gba/io.c`).
- **Found in:** Pokémon FireRed PSP, 2026-10-17.

### A reference emulator for checking frames
- `Tools/gbaref/gbaref ROM INPUT OUTDIR` (built on mGBA 0.10.5's core) runs the original ROM with
  the same `<frames> <buttons>` input file as the ports' PPSSPP tests and writes PPM screenshots,
  so a port's frame can be compared with the real game's.

### Writes and reads near address 0 crash the PSP, not the GBA
- **Seen as:** FireRed showed the copyright screen, went black and the PSP turned off, but only once
  a save existed.
- **Cause:** reading the save at boot copies the boxes through `gPokemonStoragePtr` while it's still
  NULL. On the GBA, 0x00000000-0x00003FFF is the BIOS, so the writes vanish. The wireless adapter
  check at the main menu read registers through NULL the same way.
- **Workaround:** set the pointer before the save is read (pret's Emerald does) and report no
  wireless adapter. Expect more of these: test with a save, and in PPSSPP use the interpreter (`-i`),
  which stops at the first bad read with its PC. The default JIT logs "Bad memory access detected
  and ignored" with no PC and carries on, so the emulator keeps running where a PSP crashes.
- **Found in:** Pokémon FireRed PSP, 2026-10-17.

### A division by 0 stops the game (`Exception - Breakpoint`, `break 0x7`)
- **Seen as:** a freeze in the middle of a battle move (Spark in Pokémon FireRed). Under PSPLink:
  `Exception - Breakpoint`, with the EPC on `break 0x1c00` (code 7) right after a `div`. PPSSPP
  plays the same build fine: it logs `E BREAK!` and carries on.
- **Cause:** for MIPS, GCC follows every integer division with a check that traps (`break 7`) when the
  divisor is 0. The GBA's division by 0 just gives 0, and the retail games do divide by 0 in places
  (Spark's sparks `% 0`; pokeemerald's `SAFE_DIV`/`SAFE_MOD` mark others, like the Master Ball's
  catch odds).
- **Workaround:** build with `-mno-check-zero-division`: the Allegrex's `div` by 0 doesn't trap.
  Give known cases the GBA's result with `SAFE_DIV`/`SAFE_MOD`. To find the rest, build a test
  version with the check and grep PPSSPP's log for `BREAK!`. libc and libgcc's 64-bit division
  keep their own traps.
- **Confirmed:** PSP 3001, 6.61 PRO-C, pspdev GCC 15.2; PPSSPP 1.20 doesn't stop.
- **Found in:** Pokémon FireRed PSP, 2026-10-23.

### agbcc pads structs that GCC doesn't, and saves notice
- **Seen as:** nothing in the port itself (it reads what it wrote), but saves don't move to or from a
  GBA or an emulator intact: everything after a certain field is shifted.
- **Cause:** agbcc rounds a struct containing a halfword or larger member up to a multiple of 4 bytes
  (byte-only structs stay as they are). GCC for MIPS doesn't, so a struct like
  `{ u8 flags[10]; s16 x, y; u8 a[4]; }` (18 bytes) is 20 on the GBA. In Pokémon FireRed the quest
  log's object events and the Fame Checker's entries moved the save's second half by 160 bytes.
- **Workaround:** `ALIGNED(4)` on such structs when they're saved, and a build-time
  `offsetof` check for every field whose GBA offset the decompilation notes in a comment (pret's
  `/*0x2CA0*/` annotations; one of them, `dewfordTrends`, is wrong in pokefirered). `pahole` on a
  `-g` build lists sizes and offsets. Convert saves written before the fix.
- **Confirmed:** Pokémon FireRed PSP, against save-editor offsets (rival name 0x3A4C, second Pokédex
  seen copy 0x3A18 in SaveBlock1).
- **Found in:** Pokémon FireRed PSP, 2026-10-25.

### Overflows the GBA got away with because of where variables sit
- **Seen as:** beating Brock in Pokémon FireRed: after his last line the battle never ends, and
  pressing B whites the player out with no badge.
- **Cause:** his defeat text (about 386 bytes) overruns `gDisplayedStringBattle[300]`. pret notes
  that on the GBA it lands in an unused array agbcc places right after it. GCC places static
  variables elsewhere, so it overwrote `gBattleTypeFlags`.
- **Workaround:** enlarge the buffer (512) under `UBFIX`. Search the decompilation's comments for
  "overflow" and "into" notes like this one: each is a bug that only stays harmless with agbcc's
  variable order.
- **Found in:** Pokémon FireRed PSP, 2026-10-25.
