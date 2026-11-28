---
topic: kotlin-flow-testing
category: knowledge
tags: [knowledge, kotlin-flow-testing]
updated_at: 2026-11-28T07:04:58.676673+00:00
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

- In imperative ViewModel methods that branch on current state, read the
upstream source StateFlow.value synchronously rather than a derived
StateFlow.value so state is immediately consistent without waiting for coroutine
collection, eliminating redundant advanceUntilIdle() calls in unit tests.
