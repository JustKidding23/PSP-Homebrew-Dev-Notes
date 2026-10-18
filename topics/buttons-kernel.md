# Buttons only the kernel can read

### The music note (and other system) buttons aren't visible to user-mode code
- **Seen as:** `sceCtrlReadBufferPositive` never reports the note button.
- **Workaround:** a small kernel-mode PRX, loaded into kernel memory, runs a thread that reads
  the buttons and writes them into a variable the game passes it. Stop that thread before the
  game exits. Keep a user-mode fallback combination (Pokémon Emerald PSP uses Select+Triangle)
  in case the module fails to load.
- **Confirmed:** PSP 3001, 6.61 PRO-C.
- **Found in:** Pokémon Emerald PSP, 2026-10-16 (`psp/kbuttons/`).

### A kernel thread can outlive the game and crash the XMB
- **Seen as:** quitting from the HOME menu showed the XMB's background and clock, then the PSP froze
  and turned off (and the next boot did the same once). Earlier, the theme background froze.
- **Cause:** the button thread runs in a kernel module, which can keep running after the game exits.
  It read its "keep running" flag from the game's memory, which by then belongs to the XMB, and kept
  writing the buttons into it. Stopping or unloading the module from the game during a HOME exit
  hangs on "Please wait" instead: the system holds the module manager while it waits for the game.
- **Workaround:** clear the flag in the exit callback itself (don't wait for the main loop), keep a
  stop flag in the module's own memory that `module_stop` sets and waits on, and have the thread
  check `sceKernelReferThreadStatus(gameThread)` before each access to the game's memory, ending
  itself once the game's thread is gone. Don't call `sceKernelStopModule` while quitting.
- **Confirmed:** PSP 3001, 6.61 PRO-C.
- **Found in:** Pokémon FireRed PSP, 2026-10-17 (`psp/kbuttons/kbuttons.c`, `ExitCallback` in `psp_hw.c`).
