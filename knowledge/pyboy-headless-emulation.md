---
topic: pyboy-headless-emulation
category: knowledge
tags: [knowledge, pyboy-headless-emulation]
updated_at: 2026-09-14T00:40:47.513681+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Headless-Emulation

- **Headless Initialization & Stepping**: PyBoy 2.0+ requires `window="null"`
  for headless emulation (replacing deprecated PyBoy 1.x `window_type="dummy"`).
  Worker instances can be initialized with `sound_emulated=True` and advanced
  synchronously via `pyboy.tick(1, render=True, sound=True)`.

- **Audio Emulation & Buffer Scaling**: Audio emulation requires a
  `sound_sample_rate` evenly divisible by 60 (e.g., 44100 or 48000). The output
  sound buffer is an `int8` array and must be scaled and converted to 16-bit
  signed stereo PCM for downstream audio consumers.

- **Video Framebuffer Extraction**: Screen extraction yields a 144x160x4 RGBA
  framebuffer (native 160x144 Game Boy resolution), producing exactly 69,120
  bytes of raw RGB24 video data per frame for streaming and video processing.

- **Linux Runtime & Dependencies**: Headless Linux execution requires `pyboy`,
  `pillow>=10.0.0`, `numpy`, and `pysdl2-dll`. `pysdl2-dll` bundles precompiled
  Linux x86_64 SDL2 shared libraries (`libSDL2-2.0.so`), removing the need for
  system `libsdl2-dev`. Suppress its import warning (`UserWarning: Using SDL2
  binaries from pysdl2-dll`) via `PYTHONWARNINGS="ignore:Using SDL2 binaries"`
  or `warnings.filterwarnings`. Missing Pillow raises `PyBoyDependencyError` on
  `emulator.screen.image` access.

- **Input Stuck-State Prevention**: Interrupting or cancelling execution
  mid-step or mid-batch can leave buttons stuck in a pressed state. Explicitly
  reset inputs via `release_all()` and bind joypad release calls across cancel
  handlers, frame-loop interrupts, cleanup blocks, and state machine exit
  transitions in async or manual control loops.
