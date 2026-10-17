# PSP Homebrew Dev Notes

Things a real PSP does that the documentation and emulators don't tell you, found while
porting games (Super Mario 64, Ocarina of Time, Pokémon Emerald) to a PSP 3001. Each entry
says what you'll see, why it happens, how to work around it, and how it was confirmed.

These notes are written and kept up to date with the help of an AI assistant (Claude, by
Anthropic) during that porting work. Every entry marked **confirmed on hardware** was tested
on a real PSP; anything seen only in an emulator says so.

## Topics

| File | What's in it |
|---|---|
| [topics/cpu-allegrex.md](topics/cpu-allegrex.md) | The Allegrex CPU: the `lwl`/`lwr` hazard and the GCC flags that avoid it |
| [topics/filesystem-io.md](topics/filesystem-io.md) | Memory stick files: `fopen` with relative paths, `sceIoRename`, safe saving |
| [topics/emulator-vs-hardware.md](topics/emulator-vs-hardware.md) | Where PPSSPP and a real PSP disagree |
| [topics/debugging.md](topics/debugging.md) | What a crash looks like, PSPLink, finding the crashing line |
| [topics/xmb-assets.md](topics/xmb-assets.md) | ICON0, PIC1 and SND0.AT3: making the PSP menu's art and music |
| [topics/buttons-kernel.md](topics/buttons-kernel.md) | Buttons only the kernel can read (the music note / home buttons) |

## Entry format

```
### Short name of the problem
- **Seen as:** what happens (freeze, wrong value, file missing...)
- **Cause:** why
- **Workaround:** what to do instead
- **Confirmed:** hardware model + firmware, or "emulator only"
- **Found in:** project and date
```

## Test setup

PSP 3001, firmware 6.61 PRO-C. pspdev toolchain (GCC 15.2). PPSSPP 1.20.

## Contributing

Found something else? Issues and pull requests are welcome. Please say which PSP model and
firmware you saw it on.

For AI agents and tools: [llms.txt](llms.txt) has a one-line summary of every entry.

## License

Public domain ([CC0 1.0](LICENSE)). Use it however you like, no credit needed.
