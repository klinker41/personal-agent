---
topic: llm-plays-pokemon
category: project
tags: [project, llm-plays-pokemon]
updated_at: 2026-09-18T00:01:10.070654+00:00
confidence: 0.95
---

# Project: Llm-Plays-Pokemon

## System Architecture & Toolchain

- **Core Runtime & Stack**: Bun with TypeScript, Hono web framework, Zod
  schemas, native `bun:sqlite` (WAL mode, foreign keys), native WebSockets,
  and `bun test`.
- **TypeScript Configuration**: `tsconfig.json` specifies module `Preserve`,
  moduleResolution `bundler`, and path alias `@/*` -> `./src/*` without
  deprecated `baseUrl`.
- **Python Emulation Environment**: Dedicated uv-managed virtual environment
  (`.venv` via `/usr/bin/python3`), PyBoy 2.7.0 (`window="null"`,
  `sound_emulated=True`), pillow 12.3.0, numpy 2.5.3, pysdl2 0.9.17, and
  pysdl2-dll 2.32.10. Suppresses SDL warnings via `PYTHONWARNINGS="ignore:Using
  SDL2 binaries"`. Hybrid manifest (`pyproject.toml` PEP 621 declarative and
  `requirements.txt`).
- **Media Dependencies**: Static FFmpeg binary at
  `/home/developer/.local/bin/ffmpeg` (BtbN GPL build with `drawtext`, `tee`,
  `apad`, `aresample`) and bundled TrueType fonts in `assets/fonts/` (fallback
  to `Inter-Regular.ttf` and `Inter-Bold.ttf`).
- **Container Environment**: Multi-stage Dockerfile (`oven/bun:1-debian`
  builder, standalone Bun and Python runtime). `ROMS_DIR` and `DATA_DIR` fall
  back to workspace-local paths (`./roms`, `./data`) to avoid root container
  permission requirements.

## Emulator Subsystem & IPC (`src/emulator/`)

- **Dual Unix Domain Socket IPC**:
  - *Control Socket (`/tmp/llm-pokemon-control.sock`)*: Bidirectional NDJSON RPC
    (`load_rom`, `execute_batch`, `capture_screenshot`, state management).
    Unlinks stale sockets on startup. Preserves incoming request correlation
    IDs on errors (ID 0 reserved strictly for unparseable payloads); provides
    camelCase state metrics (`totalFrames`, `currentTicks`) alongside legacy
    snake_case.
  - *Media Socket (`/tmp/llm-pokemon-media.sock`)*: Fixed 16-byte `PKMN` binary
    header streaming
<truncated 5916 bytes>
hitecture & Mock Infrastructure (`tests/`)

- **Tiered Opaque-Box Methodology**: 32+ test files executing under `bun test`
  in <30 seconds across 5 tiers:
  - *Tier 1*: Functional coverage across 35 features.
  - *Tier 2*: Boundary and edge cases (>=5 tests per domain; genuine subsystem
    execution over synthetic closures, wire-level segmented NDJSON reassembly,
    stale socket unlinking).
  - *Tier 3*: Pairwise cross-feature interactions (FSM + WS + overlays, LLM +
    emulator step + pHash + DB, gamepad lease takeover/recovery, audio silence
    + FFmpeg + RTMP tee).
  - *Tier 4*: Multi-turn real-world scenarios (cold boot, port 4401 takeover and
    resumption, loop/reprompt recovery, victory teardown).
  - *Tier 5*: Adversarial concurrency (`tier5_core_adversarial.test.ts`,
    `tier5_broadcast_web_adversarial.test.ts`): rapid 50ms lease renewals,
    input bursts, 20 concurrent WS drops, FFmpeg pipe resilience, reprompter
    exhaustion.
- **Test Mocks (`tests/mocks/`)**:
  - `MockWorkerServer`: Dual Unix domain socket bridge (NDJSON RPC + 16-byte
    PKMN frames), preserves request correlation IDs on errors, supports
    camelCase and snake_case properties, unlinks stale sockets on startup.
  - `MockLlmServer`: Protocol-accurate OpenAI Vision API mock on `Bun.serve`,
    validates schemas, returns HTTP 400 on malformed payloads, supports error
    injection and stall loops.
  - `MockFfmpegHarness`: Named FIFO drainer and overlay file validator.
  - `MockEmulatorAdapter`: In-process deterministic RGB24 frame generation and
    pure TypeScript PNG encoding via `Bun.deflateSync`.
- **Test Execution Guidelines**:
  - All test HTTP listeners bind port 4401 and serialize execution via
    `tests/helpers/port_lock.ts` (`withPort4401Lock`) to avoid `EADDRINUSE`.
  - Top-level `afterAll()` hooks must remove temporary test directories
    (`/tmp/tier*-e2e-*`) and Unix sockets to prevent disk leaks.
  - Asynchronous rejection assertions must explicitly await
    `expect(promise).rejects` to prevent vacuous passes.

- Stream video recordings in `src/broadcast/recording.ts` are formatted as
`YYYY-MM-DD_HH-mm-ss_<romname>.mkv` with date/time prefixes followed by
lowercase normalized ROM names.
- Recording collisions in `/recordings` are handled via sequential counter
suffixes (e.g., `-1`, `-2`).
