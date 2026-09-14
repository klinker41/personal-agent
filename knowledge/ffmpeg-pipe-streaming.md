---
topic: ffmpeg-pipe-streaming
category: knowledge
tags: [knowledge, ffmpeg-pipe-streaming]
updated_at: 2026-09-14T00:41:10.162027+00:00
confidence: 0.95
---

# Knowledge: Ffmpeg-Pipe-Streaming

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
