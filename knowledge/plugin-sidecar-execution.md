---
topic: plugin-sidecar-execution
category: knowledge
tags: [knowledge, sidecar, python, uv, pip, pep668]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: Plugin Sidecar Execution & Python Environment Bootstrapping

- Sidecar daemons and scheduled jobs can execute companion binaries directly by
name or relative path when the sidecar directory is prepended to `PATH` and set
as `cwd`.
- Plugin sidecars are organized as `<plugin folder>/sidecars/<sidecar
name>/sidecar.json` with accompanying executables located in the same directory.

- Sidecar managers should wait for upstream Language Server HTTP readiness and
verify the live CSRF token prior to spawning child daemon processes to prevent
unauthenticated startup failures.

- Scheduled sidecars can implement headless maintenance routines via `builtin:
schedule` and `agentapi new-conversation` using structured agent prompts without
requiring custom Python or bash scripts.
- When constructing git commit commands inside automated agent prompts or bash
commands, use multiple `-m` flags rather than `\n` escapes to properly preserve
multiline commit messages.

- Scheduled sidecars using the builtin: schedule runner with agentapi
new-conversation require a projectId specified under
sidecars.<sidecar-id>.projectId in ~/.gemini/config/config.json; omitting it
causes rpc error: project_id is required when providing project_env_config.

- Scheduled sidecars (`builtin: 'schedule'`) execute `agentapi new-conversation`
with cron expressions and command arguments defined in `sidecar.json`,
supporting explicit model selection via `--model=<flash_lite|flash|pro>`.

- Long-running sidecar processes configured with `restart_policy: "always"`
retain imported modules in memory; terminating the worker process directly
triggers `sidecar-manager.js` to respawn it with the latest code on disk without
restarting the container.

- In modern Debian/Ubuntu container environments governed by PEP 668
(externally-managed-environment), automated or programmatic pip install commands
must include the --break-system-packages flag to install dependencies outside
virtual environments.

- When installing packages via pip programmatically from within an active Python
process, calling importlib.invalidate_caches() immediately afterward ensures
module finders discover the newly installed packages without restarting the
process.
