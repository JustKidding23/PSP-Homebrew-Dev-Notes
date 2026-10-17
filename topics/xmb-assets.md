# The PSP menu's icon, background and music

`EBOOT.PBP` can carry `ICON0.PNG` (144x80), `PIC1.PNG` (480x272 background) and `SND0.AT3`
(music played while the game is selected). Pack them with pspdev's `pack-pbp`.

### SND0.AT3 that won't play
- **Seen as:** the PSP menu stays silent.
- **Cause:** the encoder output. Files from atracdenc's current development version didn't
  play, with or without an at3tool-style `smpl` loop chunk.
- **Workaround:** use [atracdenc](https://github.com/dcherednik/atracdenc) release **0.2.3**:
  - build with `-DCMAKE_CXX_FLAGS="-include cstdint"` on a current GCC
  - `atracdenc -e atrac3 -i in.wav -o out.at3`. The lowercase `.at3` selects the RIFF container.
  - fix the RIFF size field (bytes 4-7) to `filesize - 8`. 0.2.3 writes it wrong.
  - keep it under about 500 KB. 44.1 kHz stereo, ~18 s worked.
- **Confirmed:** PSP 3001, 6.61 PRO-C.
- **Found in:** Pokémon Emerald PSP, 2026-10-16 (`psp/make_art.sh`).
