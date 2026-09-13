---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-13T00:30:59.245995+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Architecture & Container Runtime
- **Container Isolation:** `jklinker/antigravity-docker:latest` runs headless
  as non-root `developer` via `gosu` with dynamic `PUID`/`PGID`, disabled
  passwordless sudo, and `umask 0002` across `conversations/`, `brain/`, and
  `annotations/`.
- **Host Execution:** Employs an isolated SSH web terminal (`ttyd`) rather than
  mounting `/var/run/docker.sock`.

## Configuration & Environment
- **Networking & Ports:** Defaults to `AGY_PORT=4400` and `AGY_HUB_PORT=4402`
  (passed via `--remote-control --hub-port` for deterministic hub discovery).
- **Environment Flags:**
  - *Auth & Host:* `RC_NAME`, `AUTH_PASSWORD`, `HOST_SSH_DIR`.
  - *Feature Toggles:* `ENABLE_IDE`, `ENABLE_TERMINAL` (default: `true`).
  - *Telemetry:* `BLOCK_TELEMETRY=true` (default; sinkholes telemetry to
    `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out variables).
- **Storage & State:**
  - Subcommand `setup` handles initial auth with mounted `~/.gemini`.
  - `entrypoint.sh` initializes empty `$GEMINI_DIR/config/projects/` and purges
    stale CSRF tokens.
  - Runtime state (`antigravity_state.pbtxt`, `installation_uuid`, migrations)
    and `cli.log` reside in `$GEMINI_DIR/antigravity-cli/`.
- **Default Policies:** `enableTerminalSandbox: true`,
  `nonWorkspaceFiles: ALLOW`, `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`, with sidebar navigation
  shortcuts for VS Code IDE and Host Terminal.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security:** 256-bit session tokens, in-memory session cleanup, IP
  rate-limiting on `/__auth/login`, 16 KB request body limit, path traversal
  protection, security headers (CSP, frame/content-type options), and
  centralized body parsing via `proxy/lib/security.js`.
- **Protocol & Reverse Proxy:** Enforces `useWebSocket=true` for `/` and
  `/c/..
<truncated 592 bytes>
models.json` with masked
  API keys.

## Translation Proxy & Transcoding
- **Activation & Routing (`proxy/translation-proxy.js`):**
  - Conditionally enabled via `entrypoint.sh` only when custom models are
    configured at container startup, ensuring standard traffic bypasses the
    proxy to minimize failure modes.
  - Model placeholders in the `M500`-`M649` range not registered in
    `modelsManager` return `null` immediately to allow built-in models to pass
    upstream to Google.
- **Argument Transcoding (`proxy/lib/transcoder.js`):**
  - `sanitizeToolCallArgs` provides provider-agnostic argument normalization
    for both streaming and unary calls.
  - Coerces stringified booleans and integers into native types.
  - Strips `ArtifactMetadata` for workspace files outside
    `.gemini/antigravity-cli/brain/` and sets synthesized
    `ArtifactMetadata.UserFacing` to `false`.
  - Defaults `Overwrite: true` on `write_to_file` calls for non-artifact paths
    when omitted by third-party models.

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Engine & Supervisor:** Authenticated `/sidecars` REST API/UI manages
  background workers and 5-field cron tasks. Coordinates restart policies,
  unified env injection (`buildSidecarEnv`), upstream Language Server polling
  (`waitForUpstream()`), and live CSRF token discovery at
  `127.0.0.1:${AGY_HUB_PORT}`.
- **Sidecar Types:**
  - *Standalone:* Defined in `~/.gemini/config/sidecars/<id>/sidecar.json` and
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - *Plugin:* Defined in `<plugin>/sidecars/<name>/sidecar.json`, namespaced as
    `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`, prepended
    `PATH`, `PLUGIN` badge, and isolated config resets.

## Testing & Quality
- **Test Suite:** Native Node test runner (`node --test tests/*.js`).
- **State Isolation:** `tests/test-sidecar-manager.js` cleans up mock
  environment variables and filesystem fixtures in `finally` blocks to prevent
  CSRF token or state leakage between suites.
