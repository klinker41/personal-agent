---
topic: offline-sync-reconciliation
category: knowledge
tags: [knowledge, offline-sync-reconciliation]
updated_at: 2026-11-26T07:04:11.674629+00:00
confidence: 0.95
---

# Knowledge: Offline-Sync-Reconciliation

- When a background worker uploads locally queued dirty records (such as Room
rows marked toBeSynced = true), clear the dirty flag using an atomic conditional
SQL UPDATE matching the uploaded snapshot values rather than an unconditional
clear, distinguishing cleanly synced rows from rows superseded by newer local
writes or already cleared by a concurrent foreground sync.
- When merging server list responses with local offline-dirty records, query the
local pending table after awaiting parallel network requests rather than before
to narrow the race window with in-flight playback updates or background sync
completions.
