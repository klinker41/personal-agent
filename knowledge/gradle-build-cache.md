---
topic: gradle-build-cache
category: knowledge
tags: [knowledge, gradle-build-cache]
updated_at: 2026-11-26T07:01:49.073325+00:00
confidence: 0.95
---

# Knowledge: Gradle-Build-Cache

- `gradle/build-cache-node` stores cache data in `/data`, reads
`/data/conf/config.yaml` (e.g., `version: 5`, fixed `targetSize` in MiB,
`anonymousLevel: "readwrite"`), and serves HTTP cache requests at
`http://<host>:5071/cache/`, which clients can enable globally via
`$GRADLE_USER_HOME/init.d/remote-cache.gradle.kts`.
- Gradle remote task cache keys normalize Linux distributions (e.g., Ubuntu vs.
Debian) and JDK patch versions (matching on major JDK version such as Temurin
21), allowing cross-container cache hits for KSP, Kotlin compilation, Lint, and
unit tests whenever `compileSdk` and `build-tools` versions match.
