---
topic: pyboy-headless-emulation
category: knowledge
tags: [knowledge, pyboy-headless-emulation]
updated_at: 2026-09-20T00:36:18.982883+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Headless-Emulation

- **Headless Initialization & Synchronous Stepping**: PyBoy 2.0+ requires
  `window="null"` (superseding 1.x `window_type="dummy"`). Step synchronously
  using `pyboy.tick(1, render=True, sound=True)`, `speed=1`, and
  `sound_emulated=True` to lock real-time pacing (~59.73 fps) and throttle CPU.
  Cold boots require initial PPU ticks before framebuffer data becomes valid.
- **A/V Extraction & Audio Conditioning**: Screen capture yields a 144x160x4
  RGBA framebuffer (native 160x144 resolution; 69,120 bytes for RGB24 video
  streaming). Set `sound_sample_rate` divisible by 60 (e.g., 44100 or 48000)
  and scale native `int8` buffers to 16-bit signed stereo PCM. Suppress static
  bursts from DC bias (+15,000) via a single-pole IIR DC blocker filter
  (`y[n] = x[n] - x[n-1] + 0.995 * y[n-1]`).
- **Input Lifecycle & Debouncing**: Call `release_all()` across cancel
  handlers, loop interrupts, cleanup blocks, and state machine exits to prevent
  stuck keys. In Game Boy Gen 1 titles, insert neutral debounce frames between
  directional inputs; sprite turning requires ~32–36 frames before directional
  movement registers.
- **Linux Runtime & Dependencies**: Requires `pyboy`, `pillow>=10.0.0`, `numpy`,
  and `pysdl2-dll`. `pysdl2-dll` supplies Linux x86_64 SDL2 binaries
  (`libSDL2-2.0.so`), removing the need for system `libsdl2-dev`. Silence
  import warnings with `PYTHONWARNINGS="ignore:Using SDL2 binaries"` or
  `warnings.filterwarnings`. Missing Pillow triggers `PyBoyDependencyError` on
  `emulator.screen.image` access.
