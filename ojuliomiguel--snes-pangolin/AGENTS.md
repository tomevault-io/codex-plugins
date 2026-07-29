# SNES Pangolin Project Instructions

Development environment for SNES homebrew using the **ca65/ld65** toolchain (cc65).

## Tech Stack & Architecture
- **CPU**: 65816 (16-bit) assembly via `.setcpu "65816"`.
- **Memory Map**: **LoROM** configuration. See [src/snes.cfg](src/snes.cfg) for memory layout.
- **Project Structure**:
    - [src/](src/): Assembly source code (`.asm`), includes (`.inc`), and linker config (`.cfg`).
    - [assets/](assets/): Pre-compiled binary data (tiles, palettes).
    - [build/](build/): Temporary object files and final `.sfc` ROM.

## Core Workflows
- **Build**: Run `make` to compile the ROM.
- **Test/Run**: Run `make run` to build and launch in the default emulator (Mesen or Snes9x).
- **Auto-Rebuild**: Run `make watch` to trigger builds on file changes.
- **Clean**: Run `make clean` to remove build artifacts.

## Coding Patterns & Conventions
- **Register Access**: Use SNES hardware registers directly (e.g., `$2100` for Screen Display, `$2116` for VRAM Address).
- **Mode Switching**:
  - `rep #$30` to switch A, X, and Y to 16-bit.
  - `sep #$20` to switch A to 8-bit.
  - Always use `.a8`, `.a16`, `.i8`, `.i16` directives to inform the assembler of the current Register sizes.
- **Initialization**:
  - Always perform a "Forced Blank" by writing `$8F` to `$2100` before updating VRAM or PPU registers during init.
  - Clear VRAM manually as shown in [src/hello.asm](src/hello.asm#L90-L98).
- **Segments**:
  - `CODE`: Logic and main loop.
  - `HEADER`: SNES ROM registration data.
  - `VECTORS`: CPU interrupt vectors (Reset, NMI, etc.).
  - `ZEROPAGE`: Fast-access variables (address `$00..$FF`).

## Key Files
- [src/hello.asm](src/hello.asm): Primary example of BG1 tilemap and Sprite (OAM) setup.
- [src/snes.cfg](src/snes.cfg): Defines the memory map essential for `ld65` linking.
- [Makefile](Makefile): Contains all build logic and emulator path detection.

## Documentation Reference
Consult the [README.md](README.md) for emulator configuration and VS Code task integration.

---
> Source: [ojuliomiguel/snes_pangolin](https://github.com/ojuliomiguel/snes_pangolin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-28 -->
