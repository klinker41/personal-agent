---
topic: node-crypto-hardening
category: knowledge
tags: [knowledge, nodejs, security, crypto, streams, path-traversal]
updated_at: 2026-10-30T18:59:00+00:00
compacted_at: 2026-10-30T18:59:00+00:00
confidence: 0.95
---

# Knowledge: Node.js Crypto, Stream & Filesystem Hardening

- To avoid timing attacks and length leakage when comparing variable-length
  secrets or passwords in Node.js, pre-hash both values with SHA-256 first and
  call `crypto.timingSafeEqual` on the equal-length digests.
- Prevent memory exhaustion DoS attacks on Node.js HTTP body parsers by
  rejecting request streams once payload size exceeds a strict limit (e.g., 16
  KB). When handling oversized payloads, detach `data` listeners, call
  `req.resume()` to drain the stream, and retain `error` listeners so a `413
  Payload Too Large` response can be delivered cleanly without socket
  destruction or unhandled stream errors.
- Accumulate stream chunks into a `Buffer` array and decode with
  `Buffer.concat(chunks).toString('utf8')` on completion to avoid corrupting
  multibyte UTF-8 characters split across chunk boundaries.
- Use `fs.openSync` and `fs.readSync` with a byte limit inside a `try...finally`
  block (ensuring `fs.closeSync`) to safely inspect large log files without
  consuming excess memory or blocking the event loop.
- When parsing JSON request bodies in Node/Bun HTTP services,
  `JSON.parse('null')` returns `null` without rejecting and
  `typeof null === 'object'`; always fall back with
  `(await req.json().catch(() => ({}))) || {}` and verify
  `rawBody !== null && typeof rawBody === 'object' && !Array.isArray(rawBody)`
  before destructuring or accessing properties.
- When verifying that a target directory is contained within a base directory,
  append the path separator (e.g., `dir.startsWith(base + path.sep)`) to avoid
  false-positive matches against sibling directories sharing a common prefix
  (e.g., `outputs_backup` matching `outputs`).
