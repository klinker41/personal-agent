---
topic: android-builds-in-antigravity
category: knowledge
tags: [android, gradle, antigravity-docker, toolchain]
updated_at: 2026-11-23T07:12:05.756245+00:00
confidence: 1.0
---

# Android-Builds-In-Antigravity

- Persistent Android toolchain lives on the NVMe volume under
/home/developer/.gemini/android-toolchain (Temurin JDK 21 & 17, Android
cmdline-tools/android CLI, platform-tools, build-tools 37.0.0, platforms
android-37.2 & android-36) with Gradle home at /home/developer/.gemini/gradle.
- Wrapper scripts in /home/developer/.gemini/antigravity-cli/bin (java, javac,
android, sdkmanager, adb, aapt2, etc.) are already on PATH in antigravity-docker
and automatically export JAVA_HOME, ANDROID_HOME, ANDROID_SDK_ROOT, and
GRADLE_USER_HOME across container restarts without requiring sudo or image
rebuilds.
- `/home/developer/.gemini/gradle/gradle.properties` enables caching, parallel
execution, and JDK 17/21 toolchain discovery
(`org.gradle.java.installations.paths`) without overriding project JVM memory
or worker limits.

- With the 8g memory limit removed from the antigravity container, custom JVM
heap/metaspace and worker caps (org.gradle.jvmargs, kotlin.daemon.jvmargs,
org.gradle.workers.max) were removed from
/home/developer/.gemini/gradle/gradle.properties while keeping
org.gradle.caching=true.

- Persistent Android toolchain is stored on the ~/.gemini NVMe volume at
/home/developer/.gemini/android-toolchain/ (Temurin JDK 21 and 17 registered via
org.gradle.java.installations.paths, plus Android SDK at
/home/developer/.gemini/android-toolchain/sdk with cmdline-tools,
platform-tools, build-tools 36.0.0/37.0.0, and platforms
android-36/android-37.2) to survive container rebuilds without Dockerfile edits.
- Wrapper scripts in /home/developer/.gemini/antigravity-cli/bin/ (already on
PATH in antigravity-docker) for java, javac, android, sdkmanager, adb, and aapt2
automatically export JAVA_HOME, ANDROID_HOME, ANDROID_SDK_ROOT, and
GRADLE_USER_HOME=/home/developer/.gemini/gradle.
- In an 8 GiB cgroup-limited container reporting 20 host CPU threads, Gradle's
default 20 workers and 4 GiB heap trigger container OOM kills (such as during
ktfmtFormat); cap memory and concurrency in
/home/developer/.gemini/gradle/gradle.properties with
org.gradle.jvmargs=-Xmx2560m, kotlin.daemon.jvmargs=-Xmx2048m, and
org.gradle.workers.max=4.

- Concurrent Gradle builds in the container can trigger Kotlin compiler
OutOfMemoryError; wait for any active workspace Gradle process to complete
before starting verification builds, and use --no-parallel if memory is
constrained.

- Antigravity's `/home/developer/.gemini/antigravity-cli/bin/java` wrapper
defaults `GRADLE_USER_HOME` to `/home/developer/.gemini/gradle` when unset, so
`GRADLE_USER_HOME=/home/developer/.gradle` must be explicitly set in the
container environment for Gradle builds to use a `/home/developer/.gradle` bind
mount.
- Do not share a single `GRADLE_USER_HOME` (`~/.gradle`) bind mount across
multiple containers (such as `antigravity` and `jenkins-android-agent`), as
Gradle stores daemon registries, PID files, and lock-managed caches expecting a
single namespace; isolate each container's `~/.gradle` directory on the host or
use an HTTP `gradle/build-cache-node` service for shared task outputs.
