---
topic: antigravity-docker
category: project
tags: [project, antigravity-docker]
updated_at: 2026-10-08T00:33:52.646240+00:00
confidence: 0.95
---

# Project: Antigravity-Docker

## Container Runtime & Isolation
- **Runtime Security:** Headless image `jklinker/antigravity-docker:latest`
  runs non-root `developer` via `gosu` (dynamic `PUID`/`PGID`, disabled
  passwordless sudo, `umask 0002` across `conversations/`, `brain/`, and
  `annotations/`). Configurations remain platform-agnostic.
- **Host Execution:** Replaces `/var/run/docker.sock` exposure with an isolated
  SSH web terminal (`ttyd` on port 7681) to execute host commands securely.

## Configuration & Environment
- **Networking & Discovery:** Exposes `AGY_PORT=4400` (auth proxy gateway) and
  `AGY_HUB_PORT=4402` (deterministic upstream discovery via `agy
  --remote-control --hub-port`). Configured via `RC_NAME`, `AUTH_PASSWORD`, and
  `HOST_SSH_DIR`.
- **Feature Flags & Privacy:** `ENABLE_IDE` (port 8080 `code-server`) and
  `ENABLE_TERMINAL` (port 7681 `ttyd`) default to `true`, injecting sidebar
  shortcuts into the UI. `BLOCK_TELEMETRY=true` (default) sinkholes Google
  telemetry to `0.0.0.0` via `/etc/hosts` and sets OpenTelemetry opt-out flags.
- **Storage, Lifecycle & Defaults:** Auth initialized via `setup` subcommand
  with mounted `~/.gemini`. `entrypoint.sh` initializes project configs under
  `$GEMINI_DIR/config/projects/` and purges stale CSRF tokens. Runtime state
  (`antigravity_state.pbtxt`, `installation_uuid`, migrations) and `cli.log`
  reside in `$GEMINI_DIR/antigravity-cli/`. Default sandbox policies:
  `enableTerminalSandbox: true`, `nonWorkspaceFiles: ALLOW`, and
  `autoExecutionPolicy: CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`.

## Auth Proxy & Gateway (`proxy/auth-proxy.js`)
- **Security & Hardening:** Employs dynamic 256-bit session tokens, in-memory
  session cleanup, IP rate-limiting on `/__auth/login`, 16 KB body limit, path
  traversal protection, security headers (CSP, frame/content-type options), and
  centrali
<truncated 1232 bytes>
ate pushes on agent turn completion to clear UI spinners.

## Translation Proxy & Transcoding
- **Activation & Routing (`proxy/translation-proxy.js`):** Native Node streaming
  transcoder on port 4405 (no LiteLLM dependency) converting Connect-RPC
  Protobuf streams to Anthropic and OpenAI endpoints. Conditionally enabled by
  `entrypoint.sh` only when custom models are configured. Unregistered
  placeholders in `M500`-`M649` return `null` immediately, routing built-in
  models (Claude, GPT-OSS) directly upstream to Google when Astra is active.
- **Argument Transcoding (`proxy/lib/transcoder.js`):** `sanitizeToolCallArgs`
  normalizes tool arguments across streaming and unary calls, coercing
  stringified booleans and integers into native types. Strips
  `ArtifactMetadata` outside `.gemini/antigravity-cli/brain/` (setting
  `UserFacing: false`) and defaults `Overwrite: true` on non-artifact
  `write_to_file` calls when omitted by third-party models.

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
