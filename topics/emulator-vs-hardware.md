# PPSSPP vs a real PSP

PPSSPP is great for a first check and for scripted tests (`PPSSPPHeadless`), but it is not
proof that something works on a PSP. Known differences:

| Area | PPSSPP | Real PSP |
|---|---|---|
| `lwl`/`lwr` result used immediately | correct value | raw aligned word (see [cpu-allegrex.md](cpu-allegrex.md)) |
| Misaligned loads/stores | allowed | can fault |
| Speed | no cache model; timing is optimistic | slower. Emerald overworld drawing: ~11 ms in PPSSPP, 17.6 ms on a PSP 3001 |
| Media Engine code | not run | runs |
| Kernel-mode PRX modules | limited | run |
| `fopen` relative paths | work | unreliable (see [filesystem-io.md](filesystem-io.md)) |
| Crash | error in the log | silent freeze, power off after ~20 s |

Tips:
- Build with `-fsanitize=alignment` (UBSan) for PPSSPP runs to list misaligned accesses that
  hardware might not allow.
- `PPSSPPHeadless` needs an **absolute** path to the EBOOT. `printf` output only appears with `-l`.
- Headless only takes a screenshot at exit. To get frames mid-run, dump the framebuffer
  (`sceDisplayGetFrameBuf`, 512-wide) to `host0:/` from the game.
