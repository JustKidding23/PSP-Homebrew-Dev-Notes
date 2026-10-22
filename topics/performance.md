# Speed on the PSP

### Stuck at 30 frames a second when one frame's work fits a refresh
- **Seen as:** a game that draws 60 frames a second in PPSSPP draws only 30 on the PSP, with
  frame skipping keeping the game's own speed. Turbo (several game frames per refresh) looks
  choppy for the same reason.
- **Cause:** a frame-pacing trap, not raw speed. Showing a frame waits for the screen's
  refresh, so work that only just misses one makes the loop take two refreshes. A clock-based
  catch-up then runs two game frames next time, which takes longer than a refresh again, and so
  on for good, even when one frame's work fits easily.
- **Workaround:** when the loop is exactly one refresh behind and the last loop's work for the
  usual number of frames fitted well within a refresh (time drawing separately: it happens once
  however many frames ran), let that lag go instead of catching up. Really slow scenes still
  catch up. See `firered-psp` src/platform/psp.c, the comment above "framesDue > speed".
- **Confirmed:** PSP 3001, 6.61 PRO-C (Pokémon FireRed PSP, 2026-10-21).

### Measuring on the PSP itself
- Run a test build over PSPLink (`usbhostfs_pc <dir>`, then `./game.prx` in pspsh) that reads
  scripted buttons from host0: and writes timings to a host0: file when the script ends. No one
  needs to hold buttons, and the numbers are the real CPU's.
- `sceKernelGetSystemTimeLow()` around each part of a scanline renderer costs about 1 µs a call,
  fine for profiling (160 lines x 6 parts).
- PPSSPP's timings mean little: its frame times were half the PSP's for the same code.

### Cheap wins found in a GBA scanline renderer (FireRed/Emerald ports)
- Don't compare all of OAM on every line: once a frame (2.4 ms a frame on the PSP).
- A window covering the whole line is no window: use its layer and effect bits for the line
  instead of a per-pixel mask (the GBA overworld keeps WIN0 over the whole screen).
- Write rows of 8 opaque 4bpp pixels without testing each pixel.
- Float divisions by constants in per-sample audio code: multiply by the reciprocal.
- **Found in:** Pokémon FireRed PSP, 2026-10-21 (overworld 20.8 -> ~10 ms a drawn frame).
