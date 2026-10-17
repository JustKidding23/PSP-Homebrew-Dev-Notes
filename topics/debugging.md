# Debugging on a real PSP

### A crash is a silent freeze, then power off
- **Seen as:** the screen freezes. About 20 s later the PSP turns itself off. No message.
- **Cause:** the PSP's normal reaction to an unhandled exception in a homebrew program.
- **Workaround:** use PSPLink (below). A kernel PRX calling `pspDebugInstallErrorHandler`
  did **not** catch anything on 6.61 PRO-C.
- **Confirmed:** PSP 3001, 6.61 PRO-C.

### PSPLink workflow
[PSPLink](https://github.com/pspdev/psplinkusb) v3.2.1 runs a program over USB and prints
crash registers. The PC side (`usbhostfs_pc`, `pspsh`) comes with pspdev.

1. Put PSPLink in `PSP/GAME/psplink/` and start it with the USB cable in. On Linux, install
   pspdev's `share/psplinkusb/50-psplink.rules` into `/etc/udev/rules.d/` so it doesn't need root.
2. `usbhostfs_pc <dir>`: that folder becomes `host0:/` on the PSP.
3. `pspsh`, then `./game.prx`. The program must be a PRX (link with pspsdk's prx specs and run
   `psp-prxgen`). Other commands: `reset`, `scrshot x.bmp`, `modinfo <uid>`, `memprot off`,
   `peekw <addr>`, `savemem <addr> <size> <file>`.
4. A crash prints `Exception - ...` with `EPC`, `RA`, `BadVAddr` and all registers. Subtract the
   module's `TextAddr` (from `modinfo`), then `psp-addr2line -f -e game.elf <addr>`.

To drive `pspsh` from scripts: `mkfifo cmd; (sleep infinity > cmd &); (pspsh < cmd > session.log 2>&1 &)`,
then `echo "<command>" > cmd`.

### Watchdog and trace (in-game)
A watchdog thread that shows a blue screen when the main loop hasn't checked in for 5 s,
plus an optional trace file written before each step, tells you where a freeze happened
without PSPLink. See Pokémon Emerald PSP's `docs/psp.md` for an implementation.
