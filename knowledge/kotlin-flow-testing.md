---
topic: kotlin-flow-testing
category: knowledge
tags: [knowledge, kotlin-flow-testing]
updated_at: 2026-11-24T07:13:08.504188+00:00
confidence: 0.95
---

# Knowledge: Kotlin-Flow-Testing

- When unit testing components that launch background collectors on a
MutableSharedFlow (especially pipelines with time-based operators like
debounce), await sharedFlow.subscriptionCount.first { it > 0 } before emitting
test values to prevent race conditions where early emissions are dropped.

- Launch background coroutines directly on an injected CoroutineScope without
overriding the context with Dispatchers.IO, and use yield() in
UnconfinedTestDispatcher unit tests so queued background work drains
deterministically.
