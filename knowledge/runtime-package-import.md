---
topic: runtime-package-import
category: knowledge
tags: [knowledge, runtime-package-import]
updated_at: 2026-08-29T11:58:19.740099+00:00
confidence: 0.95
---

# Knowledge: Runtime-Package-Import

- When installing packages via pip programmatically from within an active Python
process, calling importlib.invalidate_caches() immediately afterward ensures
module finders discover the newly installed packages without restarting the
process.
