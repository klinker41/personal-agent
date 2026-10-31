---
topic: bun-runtime-migration
category: knowledge
tags: [knowledge, bun, hono, zlib, migration]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: Bun Runtime & Hono Framework Patterns

- `Bun.password.hash` and `Bun.password.verify` natively handle bcrypt hashes
with built-in C/Zig bindings, replacing the `bcrypt` npm package and removing
the need for `node-gyp` or C++ build tools.
- Bun automatically loads `.env` files on boot, making `dotenv` redundant.
- `cron-parser@4.9.0` is zero-dependency, whereas `cron-parser` v5 introduced
`luxon` as a heavy dependency.

- Bun's Jest compatibility layer does not support `jest.mock('fs')`; use
`jest.spyOn(fs, ...)` instead when mocking filesystem functions in test suites.

- Native Bun.inflateSync can decompress raw PNG IDAT zlib streams directly to
extract scanlines for perceptual hashing and image analysis without requiring
third-party image decoding packages.

- Use Bun.inflateSync on IDAT chunks for zero-dependency PNG decompression and
normalization to raw RGB24 frames in Bun runtimes.

- Use Hono's official trimTrailingSlash() middleware rather than monkey-patching
app.fetch, which risks prematurely consuming or corrupting the incoming request
body stream.
