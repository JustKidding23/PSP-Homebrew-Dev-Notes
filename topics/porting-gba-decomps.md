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
