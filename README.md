# Berserk: Millennium Falcon Hen — Recompilation

Static ahead-of-time recompilation of **Berserk: Millennium Falcon Hen — Seima Senki no Shō**
(PlayStation 2, 2004) for modern Windows PCs.

> **Status: Early research.** Runbook: [`docs/START-HERE.md`](docs/START-HERE.md)

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
| Title | Berserk: Millennium Falcon Hen — Seima Senki no Shō |
| JP Title | ベルセルク 千年帝国の鷹篇 聖魔戦記の章 |
| Platform | PlayStation 2 |
| Developer | Yuke's |
| Publisher | Sammy |
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
PS2 disc (ISO)
    │
    ├─ 7z x + SYSTEM.CNF       identify serial (hypothesis SLPM_656.88) and boot ELF
    │
    ▼
game ELF (MIPS R5900)
    │
    ├─ ps2_analyzer            ELF + SCE symbol DB → game.toml (stubs, skip, patches)
    │
    ▼
game.toml
    │
    ├─ ps2_recomp              R5900 decode (+MMI/VU0) → C++
    │
    ▼
generated C++ + PS2Recomp runtime (GS, VU1, IOP, SPU2, CDVD, DMA, pads, memcard)
    │
    ├─ CMake build (-msse4.1)
    │
    ▼
Native executable
```

Details and the bring-up order are in [`docs/START-HERE.md`](docs/START-HERE.md).

## Tools / Frameworks

| Tool | Purpose | License |
|------|---------|---------|
| [PS2Recomp](https://github.com/ran-j/PS2Recomp) | PS2 ELF (R5900) → C++ recompiler + runtime | GPL-3.0 |
| [ps2-web-recomp](https://github.com/snuri00/ps2-web-recomp) | Reference runtime: GS raster, VU, IOP, CDVD, DMAC | MIT |
| [PCSX2](https://github.com/PCSX2/pcsx2) | Reference PS2 emulator (hardware behaviour) | GPL-2.0 |

> The PS2 has no multi-port framework equivalent to Dreamcast's `dreamcomp`. Each port writes its
> own hardware bring-up; see `docs/START-HERE.md` for what to borrow and from where.

## Building

The port builds against a `PS2Recomp` checkout (recompiler + runtime). This repository holds the
game config, extracted-symbol work and docs; the executable is produced by CMake with SSE4.1 or
better. Start with [`docs/START-HERE.md`](docs/START-HERE.md) — it also covers re-syncing the
vendored runtime, which is a September 2026 snapshot.

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
