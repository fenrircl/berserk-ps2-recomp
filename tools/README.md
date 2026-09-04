# Tools — Berserk PS2 Recompilation

Prebuilt binaries and runtime sources copied from
[**PS2Recomp**](https://github.com/ran-j/PS2Recomp) (by ran-j, GPLv3)
so they can be adapted and versioned independently for this project.

Upstream: https://github.com/ran-j/PS2Recomp
License: **GPLv3** (different from berserk-dc-recomp, which uses MIT dcrecomp)
> GPLv3 is copyleft. Any distributed build of this project must also be
> GPLv3-compatible.

## What each tool does

| Tool | Purpose |
|------|---------|
| `ps2_analyzer <ELF> <out.toml> [sce_db]` | Scans a PS2 ELF, extracts functions/symbols/relocations, writes a TOML config for the recompiler. SCE symbol database is embedded. |
| `ps2_recomp <config.toml>` | Reads TOML + ELF, decodes R5900 instructions, generates C++ source. |
| `ghidra/ExportPS2Functions.java` | Ghidra script to export per-function boundaries (optional but recommended for function map). |

### Runtime sources (`runtime/`)

Native PS2 hardware emulation that recompiled code is linked against.

| Area | Purpose |
|------|---------|
| `lib/vu/` | VU0/VU1 vector cores (upper + lower microcode) |
| `lib/Kernel/EeScheduler.cpp` | Emotion Engine thread scheduler |
| `lib/Kernel/Stubs/GS.cpp` | Graphics Synthesizer stubs |
| `lib/Kernel/Stubs/CD.cpp` | Disc/CDVDFSV access |
| `lib/Kernel/Stubs/FileIO.cpp` | File I/O |
| `lib/Kernel/Stubs/Audio.cpp` | SPU2-aware audio |
| `lib/Kernel/Stubs/Pad.cpp` | DualShock 2 input |
| `lib/Kernel/Stubs/MemoryCard.cpp` | Save data |
| `lib/Kernel/Stubs/MPEG.cpp` | FMV playback |
| `lib/Kernel/Stubs/*.cpp` | System, SIF, DMA, IPU, LibC, RPC, Deci2 ... |

## Pipeline — the exact working steps

> You must provide a legally obtained ISO dump of
> **Berserk: Millennium Falcon Hen ~Wasurebana no Shō~** (NTSC-J, JP only).
> The repo contains no game data.

### 1. Extract the ELF from the ISO

This is the only "extraction" step, and it's simpler than the Dreamcast flow:
a PS2 disc's main executable is a standard ELF at the CD root.

```bash
# Option A (if ISO mounted or extractable like a filesystem)
# root of the disc contains the ELF (starts with SLPM, SLUS, SLES...)
# Berserk Millennium Falcon's product code is SLPM-66065

# Option B — extract with 7-Zip / 7z (treats PS2 ISO as archive)
7z x berserk_pf.iso SYSTEM.CNF        # reveals BOOT2 = cdrom0:\SLPM_660.65;1
7z x berserk_pf.iso SLPM_660.65       # extract the main ELF
mv SLPM_660.65 game.elf

# Option C — programmatic ISO9660 extraction if 7zip not desired
# (see notes below)
```

Confirm the ELF is valid:
```bash
file game.elf   # should report ELF 32-bit MSB executable, MIPS R5900 / Sony PS2
```

### 2. Analyze the ELF (generate TOML)

Runs entirely offline; uses the embedded SCE symbol database.

```bash
./tools/ps2_analyzer game.elf game.toml
```

Optional but recommended for better function boundaries: run Ghidra,
then `ExportPS2Functions.java` to create a `functions.csv`, and point the
recompiler's `general.ghidra_output` at it.

### 3. Recompile ELF → C++

Write a `berserk.toml` config (see `general.*` keys in
[PS2Recomp README](https://github.com/ran-j/PS2Recomp#readme)):

```toml
[general]
input = "game.elf"
output_folder = "src/game"
# ghidra_output = "functions.csv"   # optional

[general.stubs]      # force known runtime funcs as stubs
# ... created by ps2_analyzer
```

Then:
```bash
./tools/ps2_recomp berserk.toml
```

Generates `src/game/*.cpp` + the function table and register functions.

### 4. Build the game project

Add `src/game/*.cpp` to a CMake target linked against the runtime, along
with a `main.cpp` bootstrap that:
- allocates the guest memory model (32 MB EE RAM + 4 MB VRAM)
- registers all recompiled functions
- calls the EE scheduler

Fill in the kernel stubs (GS, CD, FileIO, Audio, Pad, MemoryCard) —
currently `TODO_NAMED()` placeholders — to drive the actual game.

```bash
cmake -S . -B build
cmake --build build --config Release
```

## Known caveats

- `ps2_recomp` has NO `--help`/`--version` (treats argv[1] as the TOML path).
- Prebuilt binaries here are **Linux x86-64**; build for Windows on the
  target machine if releasing for Windows.
- Runtime uses SSE4.1 intrinsics — build `-msse4.1`+.
- The runtime GUI needs raylib/GLFW/X11 + ffmpeg/libav; headless runtime
  build skips it.
- "Experimental" upstream: static recomp is complete for the ISA; the
  hardware stubs (GS/VU/etc.) are the hard, game-specific part.

## Credits

- **ran-j** — PS2Recomp (GPLv3)
- **N64Recomp / Wiseguy** — established the static-recomp approach
- **PCSX2 team** — reference for PS2 hardware behavior