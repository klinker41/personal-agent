---
topic: ffmpeg-drawtext-overlay
category: knowledge
tags: [knowledge, ffmpeg-drawtext-overlay]
updated_at: 2026-09-14T00:39:18.432729+00:00
confidence: 0.95
---

# Knowledge: Ffmpeg-Drawtext-Overlay

- **FFmpeg Builds**: John Van Sickle static FFmpeg 7+ builds omit `libharfbuzz`
  and lack `drawtext`. Use BtbN GPL Linux static builds (`master-latest`) to
  ensure built-in support for `drawtext`, `tee`, `apad`, `aresample`, and
  `libx264` when paired with local TrueType fonts.
- **Atomic File Updates**: Dynamic text overlay updates polled by FFmpeg's
  `drawtext` filter (`reload=1` via `textfile`) must use POSIX atomic writes
  (writing to a temporary `.tmp` file and replacing via `fs.renameSync`). This
  prevents race conditions, partial reads, 0-byte reads, UTF-8 decode errors,
  file corruption, and visual text flickering or tearing.
- **Zero-Byte Handling**: Never write 0 bytes to dynamic `drawtext` overlay
  files. Output whitespace or a placeholder when empty to prevent FFmpeg
  warnings or pipeline crashes.
- **Literal '%' Handling**: Literal `%` characters in dynamic text files
  loaded via FreeType `drawtext` trigger expansion errors (`Stray % near ...`,
  exit code 234). Add `expansion=none` to filter parameters or sanitize `%`
  characters in the input.
