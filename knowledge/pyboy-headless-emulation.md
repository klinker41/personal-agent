---
topic: pyboy-headless-emulation
category: knowledge
tags: [knowledge, pyboy-headless-emulation]
updated_at: 2026-09-16T00:40:10.310352+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Headless-Emulation

- **Headless Initialization & Stepping**: PyBoy 2.0+ requires `window="null"`
  for headless emulation (replacing PyBoy 1.x `window_type="dummy"`). Set
  `sound_emulated=True` and speed to 1 for real-time wall-clock pacing
  (~59.73 fps) to avoid unthrottled CPU speed. Advance synchronously via
  `pyboy.tick(1, render=True, sound=True)`.

- **Video Framebuffer Extraction**: Screen capture yields a 144x160x4 RGBA
  framebuffer (native 160x144 Game Boy resolution), producing exactly 69,120
  bytes of raw RGB24 video per frame for streaming. Cold boots require PPU
  execution ticks before valid framebuffer data can be captured.

- **Audio Emulation & Signal Processing**: Use a `sound_sample_rate` evenly
  divisible by 60 (e.g., 44100 or 48000). The native `int8` buffer must be
  scaled and converted to 16-bit signed stereo PCM. To eliminate static bursts
  from large DC bias offsets (e.g., +15,000), filter samples through a
  single-pole IIR DC blocker (`y[n] = x[n] - x[n-1] + 0.995 * y[n-1]`).

- **Input Handling & Debouncing**: Prevent stuck buttons by explicitly calling
  `release_all()` across cancel handlers, frame-loop interrupts, cleanup
  blocks, and state machine exit transitions. For Game Boy Gen 1 games, insert
  neutral debounce frames between directional inputs because sprite turning
  requires ~32–36 frames before directional walking registers.

- **Linux Runtime & Dependencies**: Headless Linux execution requires `pyboy`,
  `pillow>=10.0.0`, `numpy`, and `pysdl2-dll`. `pysdl2-dll` bundles Linux x86_64
  SDL2 libraries (`libSDL2-2.0.so`), eliminating system `libsdl2-dev`.
  Suppress its import warning via `PYTHONWARNINGS="ignore:Using SDL2 binaries"`
  or `warnings.filterwarnings`. Missing Pillow raises `PyBoyDependencyError`
  on `emulator.screen.image` access.
