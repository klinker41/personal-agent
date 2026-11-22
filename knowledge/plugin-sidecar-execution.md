---
topic: plugin-sidecar-execution
category: knowledge
tags: [knowledge, sidecar, python, uv, pip, pep668]
updated_at: 2026-11-22T07:43:54.877656+00:00
compacted_at: 2026-11-22T07:43:54.877656+00:00
confidence: 0.95
---

# Knowledge: Plugin Sidecar Execution & Environment Bootstrapping

- **Layout & Execution Context**: Plugin sidecars live at
  `<plugin-folder>/sidecars/<sidecar-name>/sidecar.json` alongside companion
  executables. Setting the sidecar directory as `cwd` and prepending it to
  `PATH` lets daemons and scheduled jobs invoke local binaries by name or
  relative path.
- **Startup & Reload Lifecycle**: Sidecar managers must await upstream
  Language Server HTTP readiness and verify a live CSRF token before spawning
  child daemons to prevent unauthenticated startup failures. Daemons with
  `restart_policy: "always"` cache imported modules in memory; killing the
  worker process directly triggers `sidecar-manager.js` to respawn it with
  updated on-disk code without restarting the container.
- **Hub Mode Limitation**: In `agy --hub` mode, `GetServerConfiguration`
  returns `"subclientType": "hub"` with `"sidecars": {}` and disables the
  `enable-sidecars` and `scheduled-tasks-v2` feature flags, leaving the
  native Go `SidecarManager` uninitialized and returning
  `failed_precondition: sidecar manager not initialized` on `GetSidecars` and
  `SubscribeToSidecars`.
- **Environment & Tool Bootstrapping**:
  - **Python (PEP 668)**: In `externally-managed-environment` Debian/Ubuntu
    containers, programmatic or automated `pip install` calls outside
    virtual environments must pass `--break-system-packages`. After running
    `pip` inside a running Python process, call
    `importlib.invalidate_caches()` so module finders immediately discover
    new packages without a process restart.
  - **Standalone CLIs**: Sidecars can lazily install standalone CLI tools
    (e.g., `gh`) at startup into `/home/developer/.local/bin` (writable
    without `sudo` and already on `$PATH`), avoiding base
    `antigravity-docker` image rebuilds.
- **Scheduled & Maintenance Sidecars (`builtin: schedule`)**:
  - **Headless Agent Runs**: `agentapi new-conversation` prompts in
    `sidecar.json` run cron routines without custom scripts. Requires
    `sidecars.<sidecar-id>.projectId` in `~/.gemini/config/config.json`;
    omitting it causes
    `rpc error: project_id is required when providing project_env_config`.
  - **Model Selection & Testing**: `--model=<flash_lite|flash|pro>` applies
    only to the CLI invocation, leaving the persisted UI `Model Selection`
    as `None` so subsequent interactive web UI turns fall back to the
    browser's default model picker. Unit tests verifying sidecar model
    configuration must clear `ANTIGRAVITY_DEFAULT_MODEL` from the
    environment so ambient container settings do not override expected
    values.
  - **Slack Thread Continuity**: Leaving a scheduled `agentapi` conversation
    unarchived and posting a Slack webhook containing
    `https://antigravity.klinker-cabin.computer/c/<conversation_id>` lets
    `slack-chat` silently record `thread_ts -> conversation_id`, so user
    replies in that Slack thread resume the exact conversation.
  - **Local Prefiltering (`sidecar_prefilter.py`)**: Gate scheduled runs to
    short-circuit no-op checks (e.g., 0-commit git-merge checks) or execute
    deterministic maintenance (e.g., pruning dangling containers via MCP)
    locally without spawning `agentapi` LLM sessions.
  - **Log Rotation**: Overnight log-checking sidecars (e.g.,
    `jellyfin-log-checker`) should inspect at least two log files
    (`max_log_files: 2`) so midnight rotation does not hide the previous
    day's entries before early-morning checks run.
- **Git Safety in Background Sidecars**:
  - Use disposable `git worktree` checkouts (including detached worktrees
    for PR branches) on active `/workspace/<repo>` clones to avoid colliding
    with uncommitted local workspace state.
  - Automated git-merge sidecars should verify both a clean working tree and
    zero unpushed local commits before merging, and record `PRE_MERGE_HEAD`
    for clean rollback if post-merge checks fail.
  - Multiline `git commit` commands in automated agent prompts or bash
    snippets should use multiple `-m` flags rather than `\n` escapes.
