---
topic: path-traversal-containment
category: knowledge
tags: [knowledge, path-traversal-containment]
updated_at: 2026-10-14T00:02:26.908322+00:00
confidence: 0.95
---

# Knowledge: Path-Traversal-Containment

- When verifying that a target directory is contained within a base directory,
append the path separator (e.g., dir.startsWith(base + path.sep)) to avoid
false-positive matches against sibling directories sharing a common prefix
(e.g., outputs_backup matching outputs).
