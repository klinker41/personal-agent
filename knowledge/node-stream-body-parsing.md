---
topic: node-stream-body-parsing
category: knowledge
tags: [knowledge, node-stream-body-parsing]
updated_at: 2026-09-14T00:09:43.813843+00:00
confidence: 0.95
---

# Knowledge: Node-Stream-Body-Parsing

- When handling oversized HTTP request payloads, detach data listeners, call
req.resume() to drain the stream, and retain error listeners so a 413 Payload
Too Large response can be delivered cleanly without socket destruction or
unhandled stream errors.
- Accumulate stream chunks into a Buffer array and decode with
Buffer.concat(chunks).toString('utf8') on completion to avoid corrupting
multibyte UTF-8 characters split across chunk boundaries.

- When parsing JSON request bodies in Node/Bun HTTP services, payloads
containing primitive `null` produce a typeof 'object'; handlers must explicitly
verify `rawBody !== null && typeof rawBody === 'object' &&
!Array.isArray(rawBody)` before accessing properties to prevent unhandled
TypeErrors.

- When parsing HTTP JSON request bodies, using await req.json().catch(() =>
({})) is insufficient if the client sends a literal 'null' string, as
JSON.parse('null') produces null without rejecting. Always fallback with (await
req.json().catch(() => ({}))) || {} before destructuring properties.

- When parsing JSON request bodies, JSON primitive 'null' evaluates to typeof
'object'; handlers must verify `rawBody !== null && typeof rawBody === 'object'
&& !Array.isArray(rawBody)` prior to property destructuring to prevent unhandled
TypeErrors.
