---
topic: pyboy-checkpoint-restore
category: knowledge
tags: [knowledge, pyboy-checkpoint-restore]
updated_at: 2026-09-14T00:39:37.069280+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Checkpoint-Restore

- **Restoration Sequence**: When restoring PyBoy checkpoints, always call
  `loadBattery()` before `loadState()`. Cartridge battery SRAM loading triggers
  `init_pyboy()`, reinitializing the emulator and resetting `frame_count` to 0.
  Calling `loadState()` second overlays CPU registers, VRAM, and the exact
  checkpointed frame counter without being wiped or reset.

- **Decouple Battery Saves from State Snapshots**: Separate cartridge battery
  SRAM (`.sav`) from transient execution snapshots (`.state`, screenshots).
  Game Boy MBC cartridge battery SRAM is strictly 32,768 bytes (32 KB), while
  PyBoy save states are ~167 KB; passing save states into `ram_file` corrupts
  cartridge RAM. Decoupling them allows safe pruning of historical frame states
  without destroying persistent game progress.

- **File Descriptor Leak Prevention**: Wrap cartridge RAM binary data in
  `io.BytesIO` buffers rather than passing raw open file handles to
  `init_pyboy()` or RAM loaders. This prevents file descriptor leaks across
  frequent emulator resets and worker lifecycles.

- **Live Battery SRAM Extraction**: To cleanly extract cartridge battery SRAM
  without stopping an active PyBoy emulator instance, snapshot to memory via
  `pyboy.save_state(bio)`, load into a temporary headless PyBoy instance, and
  call `temp_pb.stop(save=True, ram_file=f)` to flush exactly 32 KB SRAM.

- **Sidecar Telemetry Persistence**: Persist emulator telemetry, such as
  monotonic frame counts across save states and restarts, alongside save files
  in sidecar `.meta` files to prevent telemetry desynchronization.

- **CI & Lifecycle Testing**: Use PyBoy's bundled `default_rom.gb` in automated
  test suites and CI pipelines to test emulation lifecycles, worker IPC, and
  state persistence without requiring copyrighted commercial Game Boy ROMs.
