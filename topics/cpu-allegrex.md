# The Allegrex CPU

### An `lwl`/`lwr` result used by the very next instruction is wrong
- **Seen as:** a freeze, then the PSP powering off about 20 s later. Under PSPLink: a bus
  error jumping to a junk address. The same build runs fine in PPSSPP.
- **Cause:** when an instruction uses the result of an `lwl`/`lwr` pair immediately, the PSP
  gives it the raw aligned word that `lwr` loaded, not the merged value. A moment later the
  register holds the right value, so code that uses it later (memcpy, for example) works.
  Example from Pokémon Emerald, where a function pointer at an odd address was read a byte
  at a time and GCC merged the reads:
  ```
  lwl  v1, 4(v0)
  lwr  v1, 1(v0)
  jalr v1        # jumped to 0x48030408 (the aligned word), v1 afterwards = 0x089DA048 (right)
  ```
- **Workaround:** stop GCC from making these pairs out of byte reads/writes:
  `-fdisable-tree-bswap -fno-store-merging -fno-tree-vectorize`.
  Also declare arrays with their real alignment (`__attribute__((aligned(4)))`). An
  `extern u8 REGS[]` without it turns every 32-bit access into an `lwl`/`lwr` pair.
  To check a binary, run [`../tools/lwlr_hazards.py`](../tools/lwlr_hazards.py) `game.elf`. It lists
  every `lwl`/`lwr` whose result the next instruction uses.
- **Confirmed:** PSP 3001, 6.61 PRO-C, pspdev GCC 15.2. PPSSPP 1.20 does **not** reproduce it.
- **Found in:** Pokémon Emerald PSP, 2026-10-16.
