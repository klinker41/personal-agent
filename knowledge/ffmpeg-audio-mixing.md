---
topic: ffmpeg-audio-mixing
category: knowledge
tags: [knowledge, ffmpeg-audio-mixing]
updated_at: 2026-11-28T07:03:46.738018+00:00
confidence: 0.95
---

# Knowledge: Ffmpeg-Audio-Mixing

- Pre-padding individual audio span WAVs on disk to full timeline duration
causes `O(spans * timeline_duration)` uncompressed disk bloat (e.g., ~68 GiB for
a 78-minute multi-scene movie); render spans at their natural duration instead
and apply `adelay` plus bus-level `apad`/`atrim` in-memory within FFmpeg's
filtergraph.

- Padding individual per-span temporary WAVs to the full timeline duration
causes quadratic O(N^2) temporary disk usage across long multi-scene mixes;
keeping per-span temp WAVs unpadded until stem combination and deleting them per
stem preserves bit-for-bit identical PCM output while reducing peak temp disk
usage to linear O(N) (e.g., 66.74 GiB down to 3.07 GiB across 73 scenes).

- FFmpeg's `volumedetect` filter can be parsed programmatically across rendered
scene and stitched movie streams to automatically catch silent audio tracks,
clipping, and cross-scene loudness jumps (e.g., >6 dB scene-to-scene deltas)
before publishing.
