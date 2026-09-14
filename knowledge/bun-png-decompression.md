---
topic: bun-png-decompression
category: knowledge
tags: [knowledge, bun-png-decompression]
updated_at: 2026-09-14T00:13:13.659902+00:00
confidence: 0.95
---

# Knowledge: Bun-Png-Decompression

- Native Bun.inflateSync can decompress raw PNG IDAT zlib streams directly to
extract scanlines for perceptual hashing and image analysis without requiring
third-party image decoding packages.

- Use Bun.inflateSync on IDAT chunks for zero-dependency PNG decompression and
normalization to raw RGB24 frames in Bun runtimes.
