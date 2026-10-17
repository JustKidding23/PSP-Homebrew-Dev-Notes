# Files on the memory stick

### `fopen` with a relative path doesn't create files
- **Seen as:** saves never appear on the memory stick. Works in PPSSPP.
- **Cause:** the working directory a homebrew program gets isn't reliable on hardware.
- **Workaround:** open files with `sceIoOpen` and an absolute path built from where the
  EBOOT is (`ms0:/PSP/GAME/<folder>/...`, from `argv[0]`). That also works when the program
  is started from PSPLink (`host0:/...`).
- **Confirmed:** PSP 3001, 6.61 PRO-C.
- **Found in:** Super Mario 64 PSP, 2026-10.

### `sceIoRename` won't replace an existing file
- **Seen as:** a rename onto an existing file fails.
- **Workaround:** for a safe save, write `name.tmp`, rename the old file to `name.bak`
  (remove an old `.bak` first), then rename `.tmp` to the real name. A power cut at any
  point leaves at least one good copy.
- **Confirmed:** PSP 3001, 6.61 PRO-C.
- **Found in:** Pokémon Emerald PSP, 2026-10-16.
