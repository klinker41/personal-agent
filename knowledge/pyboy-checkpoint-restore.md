---
topic: pyboy-checkpoint-restore
category: knowledge
tags: [knowledge, pyboy-checkpoint-restore]
updated_at: 2026-09-18T00:39:16.133933+00:00
confidence: 0.95
---

# Knowledge: Pyboy-Checkpoint-Restore

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
