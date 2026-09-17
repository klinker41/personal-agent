---
topic: pyboy-checkpoint-restore
category: knowledge
tags: [knowledge, pyboy-checkpoint-restore]
updated_at: 2026-09-17T00:40:16.082138+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Checkpoint-Restore

- **Restoration Sequencing & Telemetry Sync**: Pre-validate ROM presence,
  SHA256 checksums, and state integrity before terminating an active run.
  Always invoke `loadBattery()` before `loadState()`: loading battery SRAM
  triggers `init_pyboy()`, resetting the emulator and `frame_count` to 0;
  calling `loadState()` second overlays CPU registers, VRAM, and checkpointed
  frame counters without being wiped. Store emulator telemetry (such as
  monotonic frame counts across save states and restarts) in sidecar `.meta`
  files alongside save files to prevent telemetry desynchronization.

- **Decouple Battery Saves from State Snapshots**: Strictly separate cartridge
  battery SRAM (`.sav`, exactly 32 KB / 32,768 bytes for Game Boy MBCs) from
  transient execution snapshots (`.state`, ~167 KB) and screenshots. Passing
  save states into `ram_file` corrupts cartridge RAM. Decoupling enables safe
  pruning of historical frame states without losing persistent game progress.

- **In-Memory Buffering & Live SRAM Extraction**: Wrap cartridge RAM binary
  data in `io.BytesIO` buffers instead of raw file handles to prevent
  descriptor leaks across frequent resets and worker lifecycles. Cleanly
  extract battery SRAM without stopping an active instance by saving state to
  `bio = io.BytesIO()`, loading it into a temporary headless PyBoy instance,
  and calling `temp_pb.stop(save=True, ram_file=f)` to flush the 32 KB SRAM.

- **CI & Lifecycle Testing**: Use PyBoy's bundled `default_rom.gb` in automated
  test suites and CI pipelines to validate emulation lifecycles, worker IPC,
  and state persistence without relying on copyrighted commercial ROMs.
