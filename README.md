# Berserk: Millennium Falcon — Recompilation

Static ahead-of-time recompilation of **Berserk: Millennium Falcon Hen
~Wasurebana no Shō~** (PlayStation 2, 2004) for modern Windows PCs.

> **Status: Planning / Early Research**

## What this project is

A native static-AOT recompilation of the original MIPS R5900 program.

- Static MIPS R5900 to C recompilation
- Native GS (Graphics Synthesizer) renderer
- Native SPU2 audio backend
- DualShock 2 / memory card support

It is **not** a PS2 port. The original game code is translated
ahead-of-time into C, compiled natively — no emulator or BIOS required.

## Game Info

| Field | Value |
|-------|-------|
| Title | Berserk: Millennium Falcon Hen ~Wasurebana no Shō~ |
| JP Title | バーサーク ミレニアムファルコン編 ~ワスレバナノ章~ |
| Platform | PlayStation 2 |
| Developer | Yuke's |
| Publisher | Capcom |
| Release | 2004 (JP only) |
| Genre | 3D Action / Hack and Slash |
| Disc | DVD-ROM (standard PS2) |
| Region | NTSC-J |

## Hardware (PlayStation 2)

| Component | Spec |
|-----------|------|
| CPU | Emotion Engine (MIPS R5900) @ 294.912 MHz |
| GPU | Graphics Synthesizer (GS) @ 147.456 MHz |
| Sound | SPU2 (2 channels, ADPCM) |
| RAM | 32 MB (main) + 4 MB (VRAM) |

## Recompilation Pipeline

```
PS2 Disc (ISO/CUE)
    │
    ├─ ps2disc_extract.py        Extract ELF from disc
    │
    ▼
game.elf (MIPS R5900 executable)
    │
    ├─ mips_disasm.py            Find functions (prologue detection + call graph)
    │
    ▼
function_map.json
    │
    ├─ static_recompile.py       Translate MIPS R5900 → C (instruction-by-instruction)
    │
    ▼
game_code_*.c + game_functions.h + dispatch_table.c
    │
    ├─ generate_stubs.py         Fill in undefined references
    │
    ▼
Compile with CMake → Native executable
```

## Tools / Frameworks

| Tool | Purpose | License |
|------|---------|---------|
| [RecompilationStudio](https://github.com/Cybertwip/RecompilationStudio) | PS1 MIPS → C recompiler (R3000A) | Other |
| [PCSX2](https://github.com/PCSX2/pcsx2) | Reference PS2 emulator | GPLv2 |
| [DobieStation](https://github.com/PS offine/DobieStation) | Reference PS2 emulator (research) | GPLv2 |
| [N64Recomp](https://github.com/N64Recomp/N64Recomp) | MIPS recomp framework (VR4300, adaptable) | MIT |

> Note: PS2 uses MIPS R5900 (Emotion Engine), different from PS1 R3000A
> and N64 VR4300. A dedicated PS2 MIPS recompiler may need to be built
> or adapted from existing MIPS recomp frameworks.

## Building

Requirements: CMake 3.16+, C11 compiler, SDL2, Vulkan or OpenGL.

```bash
mkdir build && cd build
cmake ..
cmake --build . --config Release
```

## Project Structure

```
berserk-ps2-recomp/
├── docs/           Architecture notes, research, progress
├── tools/          Extraction and recompilation scripts
├── runtime/src/    Native hardware runtime implementations
├── tests/          Validation tests
└── README.md
```

## Legal

This is an independent preservation/research project.
Berserk: Millennium Falcon and all related names/artwork belong to
Kentaro Miura, Yuke's, Capcom, and their respective owners.

Only test with material you are legally permitted to use.
