---
topic: plugin-sidecar-execution
category: knowledge
tags: [knowledge, sidecar, python, uv, pip, pep668]
updated_at: 2026-11-02T07:31:30.393807+00:00
compacted_at: 2026-11-02T07:31:30.393807+00:00
confidence: 0.95
---

# Knowledge: Plugin Sidecar Execution & Python Environment Bootstrapping

- **Layout & Execution Context**: Plugin sidecars reside at
  `<plugin-folder>/sidecars/<sidecar-name>/sidecar.json` alongside companion
  executables. Setting the sidecar directory as `cwd` and prepending it to
  `PATH` allows daemons and scheduled jobs to invoke local binaries by name or
  relative path.
- **Startup & Reload Lifecycle**: Sidecar managers must wait for upstream
  Language Server HTTP readiness and verify a live CSRF token before spawning
  child daemons to prevent unauthenticated startup failures. Long-running
  daemons with `restart_policy: "always"` keep imported modules in memory;
  killing the worker process directly triggers `sidecar-manager.js` to respawn
  it with the latest code on disk without restarting the container.
- **Hub Mode Limitation**: In `agy --hub` mode, `GetServerConfiguration`
  returns `"subclientType": "hub"` with `"sidecars": {}` and disables the
  `enable-sidecars` and `scheduled-tasks-v2` feature flags, leaving the native
  Go `SidecarManager` uninitialized (`failed_precondition: sidecar manager not
  initialized` on `GetSidecars` and `SubscribeToSidecars`).
- **Scheduled Sidecars (`builtin: schedule`)**:
  - Execute headless maintenance routines via cron expressions and
    `agentapi new-conversation` prompts defined in `sidecar.json` without custom
    scripts.
  - Require `sidecars.<sidecar-id>.projectId` in
    `~/.gemini/config/config.json`; omitting it causes `rpc error: project_id is
    required when providing project_env_config`.
  - Support explicit model selection via `--model=<flash_lite|flash|pro>`. This
    applies only to the CLI invocation, leaving the persisted UI `Model
    Selection` as `None` so subsequent interactive turns in the web UI fall back
    to the browser's default model picker.
  - Multiline `git commit` commands in automated agent prompts or bash snippets
    should use multiple `-m` flags rather than `\n` escapes.
- **Python Environment Bootstrapping**:
  - In PEP 668 (`externally-managed-environment`) Debian/Ubuntu containers,
    programmatic or automated `pip install` commands outside virtual
    environments must pass `--break-system-packages`.
  - After installing packages via `pip` inside a running Python process, call
    `importlib.invalidate_caches()` so module finders immediately discover the
    new packages without a process restart.
