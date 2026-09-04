# Research: Berserk: Millennium Falcon (PS2)

## Disc Analysis

- **Format**: Standard PS2 DVD-ROM (NTSC-J only)
- **Executable**: ELF on disc (MIPS R5900 code)
- **Region**: NTSC-J (Japan only, never localized)
- **Product**: SLPM-65533 (Capcom)

## Known Facts from Emulation

- Playable in PCSX2 (stable)
- Uses PS2 SDK (standard library calls)
- Heavy 3D action game, similar to DC version
- FMV cutscenes: SFD (EA Packed codec on PS2)
- DVD streaming for level data

## Graphics Complexity

- 3D hack-and-slash with many enemies
- Full 3D environments
- PS2 GS (Graphics Synthesizer) rendering
- Effects: particles, lighting, shadow
- Resolution: 640x448 (standard PS2 NTSC)

## Audio

- SPU2 with ADPCM audio
- Background music + sound effects
- FMV has embedded audio

## Key Differences from DC Version

- Different engine (PS2 SDK vs Dreamcast Katana)
- Higher polygon counts possible (R5900 faster than SH-4)
- DVD-ROM allows more storage/streaming
- PS2-specific effects (motion blur, etc.)
- Japan-only release

## Key Challenges

1. **MIPS R5900 recompilation** — No existing PS2-specific recomp framework (R3000A PSX recomp exists, but R5900 is different: 128-bit SIMD, VU0/VU1 coprocessors)
2. **GS rendering** — Graphics Synthesizer is unique, different from PVR2
3. **IOP coprocessor** — PS2 I/O processor handles disc access, memory card
4. **DVD streaming** — Game streams level data from DVD during gameplay
5. **VU microcode** — Vertex/geometry processing often offloaded to VU0/VU1

## Resources

- [PCSX2 source](https://github.com/PCSX2/pcsx2) — reference PS2 emulator
- [RecompilationStudio](https://github.com/Cybertwip/RecompilationStudio) — PS1 MIPS recomp (R3000A reference)
- [N64Recomp](https://github.com/N64Recomp/N64Recomp) — MIPS recomp framework (R4300i)
- [PS2 Dev Wiki](https://ps2-dev.org/) — hardware documentation
- [PS2Dev](https://github.com/ps2dev) — PS2 SDK and tools
