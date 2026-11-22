---
topic: jenkins-job-provisioning
category: knowledge
tags: [knowledge, jenkins-job-provisioning]
updated_at: 2026-11-22T07:22:02.076198+00:00
confidence: 0.95
---

# Knowledge: Jenkins-Job-Provisioning

- Jenkins jobs created directly on disk via
<JENKINS_HOME>/jobs/<job-name>/config.xml require reloading Jenkins
configuration or restarting the Jenkins container to register cleanly in memory.
- For Android Gradle builds on persistent Jenkins agent workspaces (using
./gradlew --no-daemon), clear stale app/build/outputs/apk/release files before
assembleRelease so wildcard *.apk artifact archiving does not capture leftover
APKs, and set LogRotator artifactNumToKeep to cap retained build artifacts
independently of build log history.

- In Jenkins junit plugin >= 1465.v025001ca_dc18,
hudson.tasks.junit.JUnitResultArchiver replaced the boolean <keepLongStdio> XML
field with <stdioRetention> (values: NONE, FAILED, ALL; keepLongStdio=false maps
to NONE), and retaining <keepLongStdio> in config.xml triggers an OldDataMonitor
CannotResolveClassException warning.
- When migrating Jenkins job config.xml files directly on disk (maintaining
1000:1000 ownership and 0644 permissions), restarting the Jenkins container
reloads job configurations and clears in-memory OldDataMonitor warnings.

- Jenkins job definitions on unraid-nas (jenkins.klinker.computer) are stored at
/mnt/user/docker/jenkins/home/jobs/<job-name>/config.xml; editing config.xml
directly on disk requires restarting the jenkins container via Komodo to reload
job configurations.

- Binding nonexistent credential IDs in a Jenkins job `config.xml` causes builds
to fail immediately before running build steps; omit credential bindings until
the credentials exist in Jenkins and guard optional deployment steps in shell
scripts (`if [ -n "${SECRET_VAR:-}" ]`).
