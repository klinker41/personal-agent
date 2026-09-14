---
topic: node-crypto-hardening
category: knowledge
tags: [knowledge, node-crypto-hardening]
updated_at: 2026-09-14T00:15:57.082230+00:00
confidence: 0.95
---

# Knowledge: Node-Crypto-Hardening

- To avoid timing attacks when comparing variable-length secrets in Node.js,
hash both values with SHA-256 first and perform `crypto.timingSafeEqual` on the
fixed-length digests.
- Prevent memory exhaustion DoS attacks on Node.js HTTP body parsers by
rejecting request streams once payload size exceeds a strict limit (e.g., 16
KB).

- When performing constant-time password comparisons with
crypto.timingSafeEqual, hash both strings (e.g. SHA-256) prior to comparison to
guarantee equal buffer lengths and prevent length leakage.

- Pre-hashing passwords with SHA-256 before calling crypto.timingSafeEqual
guarantees equal-length byte buffers, preventing length-dependent timing side
channels.
