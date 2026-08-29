---
topic: plugin-sidecar-execution
category: knowledge
tags: [knowledge, plugin-sidecar-execution]
updated_at: 2026-08-29T11:56:54.608924+00:00
confidence: 0.95
---

# Knowledge: Plugin-Sidecar-Execution

- Sidecar daemons and scheduled jobs can execute companion binaries directly by
name or relative path when the sidecar directory is prepended to `PATH` and set
as `cwd`.
- Plugin sidecars are organized as `<plugin folder>/sidecars/<sidecar
name>/sidecar.json` with accompanying executables located in the same directory.
