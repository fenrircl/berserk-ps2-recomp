# START HERE — Berserk: Millennium Falcon Hen (PS2) native port

Runbook for when the ISO exists. `docs/RESEARCH.md` has the game/hardware tables.

**Set expectations first.** Dreamcast has a mature port framework (dreamcomp) that already takes a
game to a playable launcher build. **PS2 has no equivalent**: `ran-j/PS2Recomp` is a static
recompiler plus a runtime whose hardware side is still being written by each port. So this project
starts one tier lower than the Dreamcast one: first make the recompiled code boot, then make the
hardware stubs good enough for this specific game.

If the goal is "Berserk on PC with a launcher, JP audio and ES subtitles", the Dreamcast project
(`../berserk-dc-recomp`) is the shortest path. This one is the follow-up.

---

## 0. Identify the disc (before anything)

Verify on the disc, do not trust the notes:

```sh
7z l "<iso>" | head -40                      # volume, files, size
7z x "<iso>" SYSTEM.CNF -so | strings        # BOOT2 = cdrom0:\SLPM_XXX.XX;1
sha1sum "<iso>"
```

Working hypothesis, from PCSX2 scene data: **`SLPM_656.88`**, game CRC `0x9685E636`, JP-only release
(2004-10-07, Yuke's / Sammy). Confirm the serial from `SYSTEM.CNF` and correct this file if it
differs — our earlier notes said `SLPM-66065`, which is probably another title.

Also record: DVD5 vs DVD9, number of ELFs/overlays, presence of `IOPRP*.IMG` / `*.IRX` modules,
`MOVIE/*.PSS` (MPEG video), and the size of the main ELF.

## 1. Extract the executable

```sh
7z x "<iso>" SLPM_656.88 -o work/           # adjust to the real name
file work/SLPM_656.88                        # ELF 32-bit LSB, MIPS R5900
```

Keep `work/` and every derivative git-ignored (`.gitignore` already covers `*.iso`, `*.elf`, `*.bin`).

## 2. Re-sync the runtime before writing a single stub

`runtime/` here is a **September 2026 snapshot** of PS2Recomp: 83 files, ~90 `TODO_NAMED` in
`Audio.cpp`, 9 in `DMA.cpp`, 5 in `FileIO.cpp`. Upstream moved a lot since (GS backends with texture
page cache and PSMCT32/16, PSMT8/4, VU1, IOP emulator, audio/pad/CD/MPEG/IPU hooks).

```sh
git clone https://github.com/ran-j/PS2Recomp.git
git -C PS2Recomp log -1 --format='%H %ci'     # record the commit you take
```

Replace `runtime/`, `tools/ps2_analyzer`, `tools/ps2_recomp` from that commit and record it in
`docs/UPSTREAM.md` (commit SHA + what was changed locally). Keeping the lock explicit is what makes
"works on my machine" reproducible.

## 3. Recompile

```sh
./tools/ps2_analyzer work/SLPM_656.88 game.toml
./tools/ps2_recomp  game.toml            # note: ps2_recomp has no --help; argv[1] is the config
```

Then build the runtime + generated units with CMake (`-msse4.1` or better is required by the
runtime). First target is not graphics: it is a clean link and a boot log.

## 4. Bring-up order (this is where the months are)

1. **Link + boot**: generated code runs until the first unimplemented syscall. Fill syscalls in the
   order the log hits them.
2. **File I/O / CDVD**: the game streams from DVD; get `FileIO` + `CD` reading the real image before
   expecting any asset.
3. **GS**: display list → GIF packets → raster. Upstream's GS backend is the base; expect format
   gaps (palette textures, swizzles, alpha test).
4. **VU1**: microcode upload and execution. If the game skips VU1 entry, geometry will be wrong
   rather than absent — check the microcode memory writes are traced.
5. **SPU2 / audio**: `libsd` voice uploads are the usual wall. Port the working SPU2 + DualShock2 +
   CDVD implementations from `LasVola/godhand-recomp` (GPL-3.0, 3D action title, active) instead of
   writing them.
6. **FMV**: `MPEG`/`IPU` stubs; reuse `snuri00/ps2-web-recomp` (MIT — the most permissive code
   available: recompiled R5900/MMI/FPU/VU, GS raster, IOP, CDVD, DMAC).

Bring-up is judged by the report, not by hope: every unimplemented call must be named and counted.

## 5. Language (JP audio + EN/ES subtitles)

- The game is Japanese-only with Japanese voices. The English fan patch (wesker90 v1.1) proves the
  text is editable, and `SoraLeon/Berserk-Text-Editor` already extracts/inserts it — that is the
  tool to reverse for a Spanish translation.
- To confirm during survey: whether dialogue is real text (font-rendered, likely Shift-JIS or a
  custom table) or baked images (as in the Dreamcast game). Text means a translation table; images
  mean repainting, which the existing patch pipeline already does.
- Spanish accents usually need extra glyphs. Check the font atlas before promising them; if the
  font is a fixed atlas, adding `á é í ó ú ñ ¿ ¡` is a small, contained job.
- The port has no launcher layer of its own. A language selector is a runtime setting once the text
  system is understood; do not design it before step 5 of the bring-up.

## 6. Phase gates

| Phase | Done when |
|---|---|
| P0 Survey | serial, CRC, ELF list, movie format, module list recorded |
| P1 Link | generated code links and runs, first syscalls implemented, log reaches the first asset read |
| P2 Assets | file I/O + CDVD serve the real disc; textures decode |
| P3 First frame | GS renders a title screen or menu with correct palette/swizzle |
| P4 Playable | one combat scene at stable frame rate with audio and pad |
| P5 Complete | a chapter can be finished; saves work |

Anything beyond P3 is genuinely uncertain for this title — no Yuke's PS2 game has been ported yet,
and the runtime has no per-game precedent to copy. Treat estimates as ranges, not dates.

## 7. Legal

No game code, no disc data, no assets in this repo. The user supplies their own ISO. Same audit rule
as the Dreamcast project: game data never gets committed, not even a single extracted texture.
