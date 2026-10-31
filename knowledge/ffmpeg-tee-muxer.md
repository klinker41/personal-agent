---
topic: ffmpeg-tee-muxer
category: knowledge
tags: [knowledge, ffmpeg, tee-muxer, loudnorm, drawtext, streaming]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: FFmpeg Streaming, Tee Muxer & Audio/Video Filters

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

- When using FFmpeg drawtext filters driven by dynamic text files for real-time
overlays, maintain an in-memory state cache to skip redundant disk writes when
overlay content hasn't changed, significantly reducing synchronous disk I/O.

- When chaining FFmpeg audio filters with fades, place 'loudnorm' before 'afade'
(e.g., atrim -> loudnorm -> afade). Placing loudnorm after afade measures the
artificially attenuated signal and over-compensates gain to reach the integrated
loudness target.
- Standard broadcast/streaming targets for FFmpeg's loudnorm filter (EBU R128)
in spoken-word/podcasts are 'I=-16:LRA=11:TP=-1.5' (-16 LUFS integrated
loudness, 11 LU loudness range target, and -1.5 dBTP true peak ceiling).

- Linux named pipes (FIFOs) have a default buffer capacity of 64 KB (65,536
  bytes). Synchronous non-blocking writes of uncompressed video frames larger
  than 64 KB (such as 160x144 RGB24 frames at 69,120 bytes) fail with EAGAIN;
  buffered asynchronous streaming (e.g. fs.createWriteStream) is required to
  handle kernel buffer drains.
- When FFmpeg demuxes separate video and audio input pipes, pausing audio
  writes while keeping the pipe open stalls the demuxer waiting for
  interleaved audio packets because 'apad' only triggers on EOF. To prevent
  stream freezes and timestamp drift during upstream stalls or inference
  pauses, use a watchdog to inject null PCM silence at the target framerate and
  repeat the last video frame.
- Raw video input requires '-framerate <fps> -f rawvideo' prior to '-i' rather
  than '-r <fps>' to prevent libx264 non-monotonic PTS/DTS errors, while raw PCM
  audio requires '-ch_layout stereo' to prevent channel layout guessing
  warnings.
- Clock synchronization across raw pipes is stabilized using the filter chain
  'aresample=async=1000:min_hard_comp=0.100000:first_pts=0,apad', which
  eliminates microsecond clock drift, preserves PTS alignment with video, and
  ensures trailing audio padding during stalls or inference pauses.
