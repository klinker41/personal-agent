---
topic: llm-plays-pokemon
category: project
tags: [project, llm-plays-pokemon]
updated_at: 2026-09-15T00:01:32.078500+00:00
confidence: 0.95
---

# Project: Llm-Plays-Pokemon

## System Architecture & Toolchain

- Core runtime: Bun with TypeScript, Hono web framework, Zod schemas, native
  `bun:sqlite` (WAL mode, foreign keys), native WebSockets, and `bun test`.
- TypeScript config (`tsconfig.json`): module `Preserve`, moduleResolution
  `bundler`, and `@/*` -> `./src/*` path alias without deprecated `baseUrl`.
- Python emulation: uv-managed virtual environment (`.venv` via
  `/usr/bin/python3`), PyBoy 2.7.0 (`window="null"`, `sound_emulated=True`),
  pillow 12.3.0, numpy 2.5.3, pysdl2 0.9.17, and pysdl2-dll 2.32.10. Suppresses
  SDL warnings via `PYTHONWARNINGS="ignore:Using SDL2 binaries"`. Uses hybrid
  manifest (`pyproject.toml` declarative PEP 621 and `requirements.txt`).
- Media dependencies: Static FFmpeg binary at
  `/home/developer/.local/bin/ffmpeg` (BtbN GPL build with `drawtext`, `tee`,
  `apad`, `aresample`) and bundled TrueType fonts in `assets/fonts/` (fallback
  to `Inter-Regular.ttf` and `Inter-Bold.ttf`).
- Container environment: Multi-stage Dockerfile (`oven/bun:1-debian` builder,
  standalone Bun and Python runtime). `ROMS_DIR` and `DATA_DIR` fall back to
  workspace-local paths (`./roms`, `./data`) to avoid root container permission
  requirements.

## Emulator IPC & Python Worker (`src/emulator/`)

- Dual Unix domain socket IPC architecture:
  - Control socket (`/tmp/llm-pokemon-control.sock`): Bidirectional NDJSON RPC
    (`load_rom`, `execute_batch`, `capture_screenshot`, state management).
    Unlinks stale sockets on startup. Preserves incoming request IDs on errors
    (ID 0 reserved strictly for unparseable payloads); provides camelCase state
    metrics (`totalFrames`, `currentTicks`) alongside legacy snake_case.
  - Media socket (`/tmp/llm-pokemon-media.sock`): Fixed 16-byte `PKMN` binary
    header streaming 69,120-byte RGB24 (160x144) frames and PCM16 44.1 kHz
    stereo audio. Enforce
<truncated 9616 bytes>
nfigured.

## Test Architecture & Mock Infrastructure (`tests/`)

- Tiered Opaque-Box Methodology: 32+ test files executing under `bun test` in
  <30 seconds across 5 tiers:
  - Tier 1: Functional coverage across 35 features.
  - Tier 2: Boundary and edge cases (>=5 tests per domain; genuine subsystem
    execution over synthetic closures, wire-level segmented NDJSON reassembly,
    stale socket unlinking).
  - Tier 3: Pairwise cross-feature interactions (FSM + WS + overlays, LLM +
    emulator step + pHash + DB, gamepad lease takeover/recovery, audio silence
    + FFmpeg + RTMP tee).
  - Tier 4: Multi-turn real-world scenarios (cold boot, port 4401 takeover and
    resumption, loop/reprompt recovery, victory teardown).
  - Tier 5: Adversarial concurrency (`tier5_core_adversarial.test.ts`,
    `tier5_broadcast_web_adversarial.test.ts`): rapid 50ms lease renewals,
    input bursts, 20 concurrent WS drops, FFmpeg pipe resilience, reprompter
    exhaustion.
- Test Mocks (`tests/mocks/`):
  - `MockWorkerServer`: Dual Unix domain socket bridge (NDJSON RPC + 16-byte
    PKMN frames), preserves request correlation IDs on errors, supports
    camelCase and snake_case properties, unlinks stale sockets on startup.
  - `MockLlmServer`: Protocol-accurate OpenAI Vision API mock on `Bun.serve`,
    validates schemas, returns HTTP 400 on malformed payloads, supports error
    injection and stall loops.
  - `MockFfmpegHarness`: Named FIFO drainer and overlay file validator.
  - `MockEmulatorAdapter`: In-process deterministic RGB24 frame generation and
    pure TypeScript PNG encoding via `Bun.deflateSync`.
- Test Execution Guidelines:
  - All test HTTP listeners bind port 4401 and serialize execution via
    `tests/helpers/port_lock.ts` (`withPort4401Lock`) to avoid `EADDRINUSE`.
  - Top-level `afterAll()` hooks must remove temporary test directories
    (`/tmp/tier*-e2e-*`) and Unix sockets to prevent disk leaks.
  - Asynchronous rejection assertions must explicitly await
    `expect(promise).rejects` to prevent vacuous passes.

- TurnAuditLogger (`src/core/audit.ts`) records complete per-turn LLM inputs,
prompts, validation diagnostics, decisions, and emulator receipts into
`${DATA_DIR}/runs/${runId}/audit.log` (human- and LLM-readable Markdown) and
`${DATA_DIR}/runs/${runId}/audit.jsonl` (JSON Lines for programmatic analysis).
- Run checkpoint paths in `src/core/checkpoint.ts` define `auditLogPath` and
`auditJsonlPath` per run ID.
- Reprompter diagnostics in `src/llm/reprompter.ts` preserve per-attempt logs,
raw response text, diagnostic reprompts, and schema error breakdowns across all
retry cycles on both successful completions and `DecisionSchemaExhaustedError`.
- Turn audit logging is integrated into `GameController.executeTurn()` across
all lifecycle outcomes.

- Cartridge loading (/api/admin/cartridge/load) while in non-idle/non-ready
states requires calling `await controller.stop()` to finalize SQLite records,
release timers, and reset FSM error context.
- FSM transitions include 'LOADING' in COMPLETION_PENDING and use an
IDLE_TRANSITION_PATHS lookup table for transition resolution.
- Server enters standby mode when no active LLM profile is configured in the
database, preventing autonomous game loops until a profile is set up via admin
routes.

- Boot fast-forwarding (`bootSkipFrames`) defaults to 1200 frames (~20 seconds
at 60 FPS) upon ROM load before accepting model decisions.
- Post-action settling buffer (`settlingFrames`) defaults to 240 frames (4
seconds at 60 FPS) after completing an action batch before transitioning back to
THINKING mode.

- Checkpoint restore uses fail-closed pre-validation (checking ROM existence,
SHA256 integrity, run ownership, and state file validity) before terminating an
active game run.
- Checkpoint screenshot and state download endpoints enforce canonical path
containment within the data directory to prevent traversal attacks.
- Cartridge resolution (`resolveRomPathForRun`) recursively searches `romsDir`
by filename and SHA256 hash with fallback to virtual adapter paths, ensuring ROM
resolution is independent of working directory.

- PyBoy emulator worker processes and RESET RPC payloads use `--speed 1` to
enforce real-time (~59.73 fps) wall-clock pacing.
- Manual mode tracks held buttons via a `heldButtons` set and a 100ms ticker
advancing 6 frames per tick (~60fps), resetting automatically on lease timeouts,
manual button releases, or FSM transitions out of MANUAL.

- Victory claim workflow in `runTurn()` persists the claiming decision turn to
SQLite and working memory, caching claim metadata on the controller
(`latestVictoryClaim`).
- Admin victory confirmation archives the final framebuffer snapshot as durable
evidence to `/data/runs/<run_uuid>/evidence/victory.png`.

- Runtime LLM client swaps in `controller.ts` are queued via `pendingLlmClient`
during active turns and applied at turn boundaries to isolate executing turns
from mid-flight mutations.
- `loadRom()` auto-attaches the active database profile if the controller is
initialized with `UnconfiguredLlmClient`.

- Historical run inspection APIs in src/web/routes/admin.ts provide paginated
decision history, single-turn lookups, and audit log extraction
(/runs/:id/errors, /runs/:id/trace) parsing failure modes (SCHEMA_EXHAUSTED,
TRANSIENT_ERROR, STALL_HALTED) and prompt traces from audit.jsonl.

- Enforces canonical ROM file path containment in `POST /api/admin/run/start`
using `resolveCartridgePath`, failing closed with HTTP 400 on directory
traversal attempts.

- Admin CSRF origin validation (`src/web/auth.ts`) requires setting
`PUBLIC_ORIGIN` or comma-separated `ALLOWED_ORIGINS` when accessed behind a
reverse proxy.
