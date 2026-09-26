---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-09-26T00:32:06.708906+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Container Runtime & Isolation
- **Runtime Security:** Headless `jklinker/antigravity-docker:latest` runs as
  non-root `developer` via `gosu` (dynamic `PUID`/`PGID`, disabled
  passwordless sudo, `umask 0002` across `conversations/`, `brain/`, and
  `annotations/`).
- **Host Execution Gateway:** Replaces `/var/run/docker.sock` mounting with an
  isolated SSH web terminal gateway (`ttyd` on port 7681) to execute host
  commands securely without socket exposure.

## Configuration & Environment
- **Networking & Access:** Exposes `AGY_PORT=4400` (web interface/auth proxy)
  and `AGY_HUB_PORT=4402` (via `agy --remote-control --hub-port` for
  deterministic discovery). Configurable via `RC_NAME`, `AUTH_PASSWORD`, and
  `HOST_SSH_DIR`.
- **Feature Flags & Privacy:** `ENABLE_IDE` and `ENABLE_TERMINAL` (default
  `true`) provide sidebar shortcuts for VS Code and terminal.
  `BLOCK_TELEMETRY=true` sinkholes telemetry endpoints to `0.0.0.0` via
  `/etc/hosts` and sets OpenTelemetry opt-out variables.
- **Storage & Lifecycle:** Initial auth handled via `setup` subcommand with
  mounted `~/.gemini`. `entrypoint.sh` initializes
  `$GEMINI_DIR/config/projects/` and purges stale CSRF tokens. Persistent state
  (`antigravity_state.pbtxt`, `installation_uuid`, migrations) and `cli.log`
  reside in `$GEMINI_DIR/antigravity-cli/`.
- **Default Policies:** Enforces `enableTerminalSandbox: true`,
  `nonWorkspaceFiles: ALLOW`, and `autoExecutionPolicy:
  CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security & Hardening:** Enforces dynamic 256-bit session tokens, in-memory
  session cleanup, IP rate-limiting on `/__auth/login`, 16 KB body limit, path
  traversal protection, security headers (CSP, frame/content-type options), and
  centralized body parsing via `proxy/lib/security.js`.
- **Protocol
<truncated 490 bytes>
articles. Model providers are managed via `/models` UI
  (`proxy/lib/models-manager.js`), persisting masked API keys to
  `~/.gemini/config/custom_models.json`.

## Translation Proxy & Transcoding
- **Activation & Routing (`proxy/translation-proxy.js`):** Native Node.js
  streaming transcoder conditionally enabled by `entrypoint.sh` only when
  custom models are configured. Unregistered placeholder models in `M500`-`M649`
  return `null` immediately, routing built-in models (Claude, GPT-OSS)
  directly upstream to Google when Astra is active.
- **Argument Transcoding (`proxy/lib/transcoder.js`):** `sanitizeToolCallArgs`
  normalizes tool arguments across unary and streaming calls, coercing
  stringified booleans and integers into native types. Strips
  `ArtifactMetadata` outside `.gemini/antigravity-cli/brain/` (synthesizing
  `UserFacing: false`) and defaults `Overwrite: true` on non-artifact
  `write_to_file` calls when omitted by third-party models.

## Sidecar Management (`proxy/sidecar-manager.js`)
- **Supervisor Engine:** Authenticated `/sidecars` REST API and UI manages
  background workers and 5-field cron tasks. Coordinates restart policies,
  unified env injection (`buildSidecarEnv`), upstream Language Server polling
  (`waitForUpstream()`), and live CSRF token discovery at
  `127.0.0.1:${AGY_HUB_PORT}`.
- **Sidecar Types:**
  - *Standalone:* Defined in `~/.gemini/config/sidecars/<id>/sidecar.json` and
    toggled via `sidecars[id].enabled` in `~/.gemini/config/config.json`.
  - *Plugin:* Defined in `<plugin>/sidecars/<name>/sidecar.json`, namespaced as
    `<plugin-name>/<sidecar-name>`, executed with isolated `cwd`, prepended
    `PATH`, `PLUGIN` UI badge, and isolated config resets.

## Testing & Quality
- **Test Suite & Isolation:** Native Node test runner
  (`node --test tests/*.js`). Maintains state isolation by cleaning up mock
  environment variables and filesystem fixtures in `finally` blocks (e.g.,
  `tests/test-sidecar-manager.js`) to prevent CSRF token or state leakage
  between test suites.
