---
topic: bun-runtime-migration
category: knowledge
tags: [knowledge, bun-runtime-migration]
updated_at: 2026-09-06T00:00:56.634890+00:00
confidence: 0.95
---

# Knowledge: Bun-Runtime-Migration

- `Bun.password.hash` and `Bun.password.verify` natively handle bcrypt hashes
with built-in C/Zig bindings, replacing the `bcrypt` npm package and removing
the need for `node-gyp` or C++ build tools.
- Bun automatically loads `.env` files on boot, making `dotenv` redundant.
- `cron-parser@4.9.0` is zero-dependency, whereas `cron-parser` v5 introduced
`luxon` as a heavy dependency.
