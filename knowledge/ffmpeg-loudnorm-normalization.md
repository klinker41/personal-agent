---
topic: ffmpeg-loudnorm-normalization
category: knowledge
tags: [knowledge, ffmpeg-loudnorm-normalization]
updated_at: 2026-09-10T00:01:39.734106+00:00
confidence: 0.95
---

# Knowledge: Ffmpeg-Loudnorm-Normalization

- When chaining FFmpeg audio filters with fades, place 'loudnorm' before 'afade'
(e.g., atrim -> loudnorm -> afade). Placing loudnorm after afade measures the
artificially attenuated signal and over-compensates gain to reach the integrated
loudness target.
- Standard broadcast/streaming targets for FFmpeg's loudnorm filter (EBU R128)
in spoken-word/podcasts are 'I=-16:LRA=11:TP=-1.5' (-16 LUFS integrated
loudness, 11 LU loudness range target, and -1.5 dBTP true peak ceiling).
