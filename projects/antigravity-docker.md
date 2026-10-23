---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-10-23T06:30:32.610214+00:00
compacted_at: 2026-10-23T06:30:32.610214+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Container Runtime & Isolation
- **Runtime Security:** Headless image `jklinker/antigravity-docker:latest` runs
  non-root `developer` via `gosu` (dynamic `PUID`/`PGID`, disabled passwordless
  sudo, `umask 0002` across `conversations/`, `brain/`, and `annotations/`).
  Platform-agnostic isolation replaces `/var/run/docker.sock` mounting with an
  SSH web terminal (`ttyd` on port 7681 via `HOST_SSH_DIR`) for host commands.

## Configuration & Environment
- **Networking & Services:** Exposes `AGY_PORT=4400` (auth proxy gateway),
  `AGY_HUB_PORT=4402` (hub discovery via `agy --remote-control --hub-port`), and
  port 4405 (translation proxy). Configured via `RC_NAME` and `AUTH_PASSWORD`.
  Auxiliary services `ENABLE_IDE` (port 8080 `code-server`) and
  `ENABLE_TERMINAL` (port 7681 `ttyd`) default to `true` with UI shortcuts.
- **Privacy & Telemetry:** `BLOCK_TELEMETRY=true` (default) sinkholes Google
  telemetry to `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out flags.
- **Storage & Sandboxing Lifecycle:** Auth initialized via `setup` subcommand
  with mounted `~/.gemini`. `entrypoint.sh` initializes project configs under
  `$GEMINI_DIR/config/projects/` and purges stale CSRF tokens. Runtime state
  (`antigravity_state.pbtxt`, `installation_uuid`, migrations) and `cli.log`
  reside in `$GEMINI_DIR/antigravity-cli/`. Sandbox defaults:
  `enableTerminalSandbox: true`, `nonWorkspaceFiles: ALLOW`, and
  `autoExecutionPolicy: CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security & Protocol Proxy:** Hardened via `proxy/lib/security.js` with
  dynamic 256-bit session tokens, in-memory cleanup, IP rate-limiting on
  `/__auth/login`, 16 KB body limit, path traversal defense, and security
  headers. Enforces `useWebSocket=true` on root and `/c/...` routes, sets
  `X-Accel-Buffering: no`, flushes headers immediately, preserves gRPC
  streaming headers (`TE: trailers`, `Trailer`, `grpc-status`), and suppresses
  upstream TCP RST packets on socket cleanup.
- **Endpoints, UI & Mobile Gateway:** Exposes unauthenticated `/status` health
  check (`200`/`503`), cosmic glassmorphic UI (`renderPageLayout`), and
  `/models` provider manager (`~/.gemini/config/custom_models.json`). Mobile
  Gateway (`/gateway/v1`) uses bearer token auth decoupled from browser cookies
  with hybrid polling (60s passive, 2s burst for 10s) and immediate turn
  completion pushes to clear UI spinners.

## Translation Proxy & Transcoding
- **Activation & Routing (`proxy/translation-proxy.js`):** Native Node streaming
  transcoder on port 4405 (no LiteLLM) translating Connect-RPC Protobuf to
  Anthropic and OpenAI endpoints, intercepting API traffic between the daemon
  and Google backend to map model tiers. Conditionally enabled by
  `entrypoint.sh` only when custom models are configured. Placeholders
  `M500`-`M649` return `null` immediately, routing built-in models (Claude,
  GPT-OSS) upstream to Google when Astra is active.
- **Argument Transcoding (`proxy/lib/transcoder.js`):** `sanitizeToolCallArgs`
  normalizes tool arguments across streaming and unary calls, coercing
  stringified booleans and integers into native types. Strips
  `ArtifactMetadata` outside `.gemini/antigravity-cli/brain/` (forcing
  `UserFacing: false`) and defaults `Overwrite: true` on non-artifact
  `write_to_file` calls when omitted.

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Supervisor Engine:** Authenticated `/sidecars` REST API and UI manages
  background workers and 5-field cron tasks. Coordinates restart policies,
  unified env injection (`buildSidecarEnv`), upstream Language Server polling
  (`waitForUpstream()`), and live CSRF token discovery at
  `127.0.0.1:${AGY_HUB_PORT}`.
- **Sidecar Types:**
  - *Standalone:* Defined in `~/.gemini/config/sidecars/<id>/sidecar.json`,
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - *Plugin:* Defined in `<plugin>/sidecars/<name>/sidecar.json`, namespaced as
    `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`, prepended
    `PATH`, `PLUGIN` UI badge, and isolated config resets.

## Testing & Quality
- **Test Suite & State Isolation:** Native Node test runner (`node --test
  tests/*.js`). Maintains state isolation by cleaning up mock environment
  variables and filesystem fixtures in `finally` blocks (e.g.,
  `tests/test-sidecar-manager.js`) to prevent CSRF token or state leakage
  between test suites.
