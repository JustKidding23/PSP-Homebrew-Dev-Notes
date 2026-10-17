# Buttons only the kernel can read

### The music note (and other system) buttons aren't visible to user-mode code
- **Seen as:** `sceCtrlReadBufferPositive` never reports the note button.
- **Workaround:** a small kernel-mode PRX, loaded into kernel memory, runs a thread that reads
  the buttons and writes them into a variable the game passes it. Stop that thread before the
  game exits. Keep a user-mode fallback combination (Pokémon Emerald PSP uses Select+Triangle)
  in case the module fails to load.
- **Confirmed:** PSP 3001, 6.61 PRO-C.
- **Found in:** Pokémon Emerald PSP, 2026-10-16 (`psp/kbuttons/`).
