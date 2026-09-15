---
topic: pyboy-checkpoint-restore
category: knowledge
tags: [knowledge, pyboy-checkpoint-restore]
updated_at: 2026-09-15T00:38:52.739353+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Checkpoint-Restore

- **Safe Restoration Workflow & Sequence**: Pre-validate all prerequisites (ROM
  presence, SHA256 checksum, and state validity) before stopping an active
  emulation run to avoid unrecoverable state loss. When restoring checkpoints,
  always call `loadBattery()` before `loadState()`. Battery SRAM loading
  triggers `init_pyboy()`, reinitializing the emulator and resetting
  `frame_count` to 0; calling `loadState()` second overlays CPU registers,
  VRAM, and the exact checkpointed frame counter without being wiped.

- **Decouple Battery Saves from State Snapshots**: Strictly separate cartridge
  battery SRAM (`.sav`, exactly 32 KB / 32,768 bytes for Game Boy MBCs) from
  transient execution snapshots (`.state`, ~167 KB, screenshots). Passing save
  states into `ram_file` corrupts cartridge RAM. Decoupling them allows safe
  pruning of historical frame states without losing persistent game progress.

- **Live Battery SRAM Extraction**: Cleanly extract cartridge battery SRAM
  without stopping an active emulator instance by saving the state to memory
  via `pyboy.save_state(bio)`, loading it into a temporary headless PyBoy
  instance, and calling `temp_pb.stop(save=True, ram_file=f)` to flush the
  32 KB SRAM.

- **File Descriptor Leak Prevention**: Wrap cartridge RAM binary data in
  `io.BytesIO` buffers rather than passing raw open file handles to
  `init_pyboy()` or RAM loaders, preventing descriptor leaks across frequent
  emulator resets and worker lifecycles.

- **Sidecar Telemetry Persistence**: Store emulator telemetry (e.g., monotonic
  frame counts across save states and restarts) in sidecar `.meta` files
  alongside save files to prevent telemetry desynchronization.

- **CI & Lifecycle Testing**: Use PyBoy's bundled `default_rom.gb` in automated
  test suites and CI pipelines to validate emulation lifecycles, worker IPC,
  and state persistence without relying on copyrighted commercial ROMs.
