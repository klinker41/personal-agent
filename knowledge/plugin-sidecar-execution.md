---
topic: plugin-sidecar-execution
category: knowledge
tags: [knowledge, sidecar, python, uv, pip, pep668]
updated_at: 2026-11-21T07:42:30.660268+00:00
compacted_at: 2026-11-21T07:42:30.660268+00:00
confidence: 0.95
---

# Knowledge: Plugin Sidecar Execution & Python Environment Bootstrapping

- **Layout & Execution Context**: Plugin sidecars reside at
  `<plugin-folder>/sidecars/<sidecar-name>/sidecar.json` alongside companion
  executables. Setting the sidecar directory as `cwd` and prepending it to
  `PATH` lets daemons and scheduled jobs invoke local binaries by name or
  relative path.
- **Startup & Reload Lifecycle**: Sidecar managers must wait for upstream
  Language Server HTTP readiness and verify a live CSRF token before spawning
  child daemons to prevent unauthenticated startup failures. Long-running
  daemons with `restart_policy: "always"` cache imported modules in memory;
  killing the worker process directly triggers `sidecar-manager.js` to
  respawn it with the latest on-disk code without restarting the container.
- **Hub Mode Limitation**: In `agy --hub` mode, `GetServerConfiguration`
  returns `"subclientType": "hub"` with `"sidecars": {}` and disables the
  `enable-sidecars` and `scheduled-tasks-v2` feature flags, leaving the
  native Go `SidecarManager` uninitialized, returning
  `failed_precondition: sidecar manager not initialized` on `GetSidecars` and
  `SubscribeToSidecars`.
- **Scheduled & Maintenance Sidecars (`builtin: schedule`)**:
  - Run headless cron routines via `agentapi new-conversation` prompts in
    `sidecar.json` without custom scripts. Requires
    `sidecars.<sidecar-id>.projectId` in `~/.gemini/config/config.json`;
    omitting it causes
    `rpc error: project_id is required when providing project_env_config`.
  - `--model=<flash_lite|flash|pro>` applies only to the CLI invocation,
    leaving the persisted UI `Model Selection` as `None` so subsequent
    interactive web UI turns fall back to the browser's default model picker.
  - Deterministic maintenance tasks (e.g., pruning dangling containers via
    MCP) can execute directly inside `sidecar_prefilter.py` to avoid spawning
    LLM agent sessions on routine runs.
  - Overnight log-checking sidecars (e.g., `jellyfin-log-checker`) should
    inspect at least two log files (`max_log_files: 2`) so midnight log
    rotation does not hide the previous day's entries before early-morning
    checks run.
  - Multiline `git commit` commands in automated agent prompts or bash
    snippets should use multiple `-m` flags rather than `\n` escapes.
- **Python Environment Bootstrapping**: In PEP 668
  (`externally-managed-environment`) Debian/Ubuntu containers, programmatic
  or automated `pip install` commands outside virtual environments must pass
  `--break-system-packages`. After installing packages via `pip` inside a
  running Python process, call `importlib.invalidate_caches()` so module
  finders immediately discover new packages without a process restart.
