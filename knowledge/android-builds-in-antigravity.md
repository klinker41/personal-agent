---
topic: android-builds-in-antigravity
category: knowledge
tags: [android, gradle, antigravity-docker, toolchain]
updated_at: 2026-11-28T07:49:54.426919+00:00
compacted_at: 2026-11-28T07:49:54.426919+00:00
confidence: 1.0
---

# Android Builds in Antigravity

## Toolchain & Environment

- The persistent Android toolchain lives on the `~/.gemini` NVMe volume at
  `/home/developer/.gemini/android-toolchain/` so it survives container
  restarts and rebuilds without `sudo` or Dockerfile edits:
  - Temurin JDK 21 and 17 (registered via
    `org.gradle.java.installations.paths`).
  - Android SDK at `/home/developer/.gemini/android-toolchain/sdk` with
    `cmdline-tools` (`android` CLI), `platform-tools`, `build-tools`
    36.0.0/37.0.0, and platforms `android-36`/`android-37.2`.
- Wrapper scripts in `/home/developer/.gemini/antigravity-cli/bin/` (already
  on `PATH` in `antigravity-docker`) for `java`, `javac`, `android`,
  `sdkmanager`, `adb`, `aapt2`, etc. automatically export `JAVA_HOME`,
  `ANDROID_HOME`, `ANDROID_SDK_ROOT`, and
  `GRADLE_USER_HOME=/home/developer/.gemini/gradle`.

## Gradle Home & Container Isolation

- The `/home/developer/.gemini/antigravity-cli/bin/java` wrapper defaults
  `GRADLE_USER_HOME` to `/home/developer/.gemini/gradle` when unset;
  explicitly set `GRADLE_USER_HOME=/home/developer/.gradle` in the container
  environment to use a `/home/developer/.gradle` bind mount instead.
- Never share a single `GRADLE_USER_HOME` (`~/.gradle`) bind mount across
  multiple containers (such as `antigravity` and `jenkins-android-agent`):
  Gradle stores daemon registries, PID files, and lock-managed caches
  expecting a single namespace. Isolate each container's `~/.gradle` on the
  host, or run an HTTP `gradle/build-cache-node` for shared task outputs.

## Gradle Configuration & Resource Tuning

- `/home/developer/.gemini/gradle/gradle.properties` enables build caching
  (`org.gradle.caching=true`), parallel execution, and JDK 17/21 toolchain
  discovery (`org.gradle.java.installations.paths`) without overriding
  project JVM memory or worker limits. Custom JVM heap/metaspace and worker
  caps (`org.gradle.jvmargs`, `kotlin.daemon.jvmargs`, and
  `org.gradle.workers.max`) were removed once the 8 GiB memory limit was
  removed from the `antigravity` container.
- Avoid concurrent Gradle builds in the container, which can trigger Kotlin
  compiler `OutOfMemoryError`; wait for any active workspace Gradle process
  to complete before starting verification builds, and pass `--no-parallel`
  when memory is constrained.
- Properties in `$GRADLE_USER_HOME/gradle.properties` override a repository's
  `gradle.properties`, allowing high workstation/server settings (`-Xmx12g`,
  `kotlin.daemon.jvm.options=-Xmx6g`) in version control while applying
  constrained limits (`-Xmx4096m`, `-Xmx2g`, `org.gradle.workers.max=4`) on
  4-vCPU / 16 GB GitHub Actions runners.
- In an 8 GiB cgroup-limited container reporting 20 host CPU threads,
  Gradle's default 20 workers and 4 GiB heap trigger OOM kills (e.g., during
  `ktfmtFormat`); in that setup, cap `gradle.properties` with
  `org.gradle.jvmargs=-Xmx2560m`, `kotlin.daemon.jvmargs=-Xmx2048m`, and
  `org.gradle.workers.max=4`.
- On a 20-thread hybrid CPU (8 P-cores + 12 E-cores), capping
  `org.gradle.workers.max=10` halves peak system load average during
  single-module Android builds without regressing wall-clock build time.
