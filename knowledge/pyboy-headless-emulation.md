---
topic: pyboy-headless-emulation
category: knowledge
tags: [knowledge, pyboy-headless-emulation]
updated_at: 2026-09-21T00:36:09.045156+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Headless-Emulation

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
