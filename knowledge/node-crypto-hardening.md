---
topic: node-crypto-hardening
category: knowledge
tags: [knowledge, node-crypto-hardening]
updated_at: 2026-08-29T11:57:15.490175+00:00
confidence: 0.95
---

# Knowledge: Node-Crypto-Hardening

- To avoid timing attacks when comparing variable-length secrets in Node.js,
hash both values with SHA-256 first and perform `crypto.timingSafeEqual` on the
fixed-length digests.
- Prevent memory exhaustion DoS attacks on Node.js HTTP body parsers by
rejecting request streams once payload size exceeds a strict limit (e.g., 16
KB).
