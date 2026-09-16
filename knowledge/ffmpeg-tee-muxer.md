---
topic: ffmpeg-tee-muxer
category: knowledge
tags: [knowledge, ffmpeg-tee-muxer]
updated_at: 2026-09-16T00:01:02.937667+00:00
confidence: 0.95
---

# Knowledge: Ffmpeg-Tee-Muxer

- In FFmpeg multi-destination RTMP tee muxers, prefix slave outputs with
[f=flv:onfail=ignore] so a drop or failure on a single RTMP target does not
crash the entire broadcast.

- Multi-RTMP output broadcasting via FFmpeg `-f tee` supports `[onfail=ignore]`
slave options to isolate stream endpoint disconnects, requiring pipe characters
in target URLs to be escaped.

- Configuring [f=flv:onfail=ignore] on each destination in FFmpeg's tee muxer
isolates endpoint failures so a single dropped RTMP target does not terminate
the broadcast, with fallback to -f null /dev/null when no outputs are
configured.

- Multi-target RTMP broadcasting using '-f tee' should specify
'[f=flv:onfail=ignore]' on individual target URLs to prevent single endpoint
disconnects from crashing the stream, backed by a null sink fallback.

- Using '-f tee' with slave syntax
'[f=flv:onfail=ignore]url1|[f=flv:onfail=ignore]url2' enables multi-destination
RTMP broadcast where individual endpoint connection failures do not crash the
primary pipeline.
- When streaming destinations are empty, directing output to a null sink or
local test stream preserves pipeline lifecycle without exiting or dropping the
ingest loop.

- Multi-destination broadcasting via a single-pass FFmpeg `-f tee` pipeline
requires adhering to the most restrictive endpoint codec; standard Twitch RTMP
requires H.264 (AVC) and rejects H.265 (HEVC).
