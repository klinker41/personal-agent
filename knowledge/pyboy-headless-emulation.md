---
topic: pyboy-headless-emulation
category: knowledge
tags: [knowledge, pyboy, pokemon, emulation, checkpoint]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: PyBoy Headless Emulation & Checkpoints

- **Headless Initialization & Stepping**: PyBoy 2.0+ requires `window="null"`
  (replaces 1.x `window_type="dummy"`). Step synchronously with
  `pyboy.tick(1, render=True, sound=True)`, `speed=1`, and
  `sound_emulated=True` to lock real-time pacing (~59.73 fps) and throttle CPU.
  Cold boots require initial PPU ticks before framebuffer data is valid.
- **A/V Extraction & Audio Conditioning**: Screen capture yields a 144x160x4
  RGBA framebuffer (native 160x144; 69,120 bytes for RGB24 streaming). Set
  `sound_sample_rate` divisible by 60 (e.g., 44100 or 48000) and scale native
  `int8` buffers to signed 16-bit stereo PCM. Eliminate DC bias (+15,000)
  static bursts via a single-pole IIR DC blocker filter:
  `y[n] = x[n] - x[n-1] + 0.995 * y[n-1]`.
- **Input Lifecycle & Debouncing**: Always call `release_all()` in cleanup
  blocks, loop interrupts, cancel handlers, and state exits to avoid stuck
  keys. In Gen 1 titles, insert neutral debounce frames between directional
  inputs; sprite turns require ~32–36 frames before directional movement
  registers.
- **Dependencies & Linux Runtime**: Requires `pyboy`, `pillow>=10.0.0`,
  `numpy`, and `pysdl2-dll`. `pysdl2-dll` bundles Linux x86_64 SDL2
  (`libSDL2-2.0.so`), bypassing system `libsdl2-dev`. Suppress import
  warnings with `PYTHONWARNINGS="ignore:Using SDL2 binaries"` or
  `warnings.filterwarnings`. Missing Pillow raises `PyBoyDependencyError` on
  `emulator.screen.image`.

- **Restoration Sequencing & Telemetry**: Pre-validate ROM presence, SHA256
  checksums, and state integrity before terminating an active run. Always
  invoke `loadBattery()` before `loadState()`: loading battery SRAM triggers
  `init_pyboy()`, resetting the emulator and `frame_count` to 0, whereas
  `loadState()` safely overlays CPU registers, VRAM, and checkpointed
  counters. Store monotonic frame counts and telemetry in sidecar `.meta`
  files alongside saves to prevent telemetry desynchronization.

- **Save Decoupling & In-Memory SRAM Extraction**: Decouple persistent
  cartridge SRAM (`.sav`, exactly 32 KB / 32,768 bytes for MBCs) from
  transient execution snapshots (`.state`, ~167 KB) and screenshots to allow
  pruning without losing game progress; passing states to `ram_file`
  corrupts cartridge RAM. Buffer RAM binary data in `io.BytesIO` to avoid
  descriptor leaks across worker cycles. Extract live SRAM without stopping
  active runs by dumping state to `io.BytesIO()`, loading it into a temporary
  headless PyBoy instance, and calling `temp_pb.stop(save=True, ram_file=f)`.

- **Automated Lifecycle Testing**: Use PyBoy's bundled `default_rom.gb` in CI
  and automated test suites to validate emulation lifecycles, worker IPC, and
  persistence without requiring copyrighted commercial ROMs.

- gen1recomp is a from-scratch Lua and LÖVE2D engine recreation of Generation 1
and 2 Pokémon rather than a binary recompilation; it reads Game Boy ROMs solely
on initial startup to dump assets, tables, and sound programs into local Lua
scripts and PNGs.
