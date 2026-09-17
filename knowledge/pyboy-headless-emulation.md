---
topic: pyboy-headless-emulation
category: knowledge
tags: [knowledge, pyboy-headless-emulation]
updated_at: 2026-09-17T00:41:04.816347+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Headless-Emulation

- **Headless Initialization & Stepping**: PyBoy 2.0+ requires `window="null"`
  for headless execution (superseding PyBoy 1.x `window_type="dummy"`). Set
  `sound_emulated=True` and speed to 1 to lock real-time wall-clock pacing
  (~59.73 fps) and prevent unthrottled CPU execution. Advance synchronously
  via `pyboy.tick(1, render=True, sound=True)`.

- **Video Framebuffer Extraction**: Screen capture yields a 144x160x4 RGBA
  framebuffer (native 160x144 Game Boy resolution), producing exactly 69,120
  bytes of raw RGB24 video per frame for streaming. Cold boots require initial
  PPU execution ticks before valid framebuffer data is available.

- **Audio Emulation & Signal Conditioning**: Use a `sound_sample_rate` evenly
  divisible by 60 (e.g., 44100 or 48000). Convert and scale the native `int8`
  buffer to 16-bit signed stereo PCM. To eliminate static bursts from large DC
  bias offsets (e.g., +15,000), filter samples through a single-pole IIR DC
  blocker (`y[n] = x[n] - x[n-1] + 0.995 * y[n-1]`).

- **Input State Lifecycle & Debouncing**: Prevent stuck buttons by invoking
  `release_all()` across cancel handlers, loop interrupts, cleanup blocks, and
  state machine exits. For Game Boy Gen 1 games, insert neutral debounce
  frames between directional inputs because sprite turning requires ~32–36
  frames before directional movement registers.

- **Linux Runtime & Environment Dependencies**: Headless Linux execution
  requires `pyboy`, `pillow>=10.0.0`, `numpy`, and `pysdl2-dll`. `pysdl2-dll`
  bundles Linux x86_64 SDL2 binaries (`libSDL2-2.0.so`), removing the need
  for system `libsdl2-dev`. Silence its import warning via
  `warnings.filterwarnings` or `PYTHONWARNINGS="ignore:Using SDL2 binaries"`.
  Missing Pillow raises `PyBoyDependencyError` on `emulator.screen.image`
  access.
