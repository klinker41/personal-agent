---
topic: google-play-publishing
category: knowledge
tags: [knowledge, google-play-publishing]
updated_at: 2026-11-22T07:22:02.076347+00:00
confidence: 0.95
---

# Knowledge: Google-Play-Publishing

- Modern `keytool` (JDK 9+) creates PKCS12 keystores by default even when given
a `.jks` filename and uses a single password without prompting for a separate
key password, so `ANDROID_KEY_PASSWORD` must be set to the same value as
`ANDROID_KEYSTORE_PASSWORD`.
- Google Play Android Developer API service accounts require 'View app
information and download bulk reports (read-only)' and 'Release apps to testing
tracks' permissions in Play Console; API permission propagation can take up to
24 hours, though saving a minor store listing edit often triggers immediate
sync.
- Apps published to Google Play must remove
`android.permission.REQUEST_INSTALL_PACKAGES` and any in-app APK self-updater
unless package installation is a permitted core function.
- Call `.strip` on credential path environment variables like
`ENV['PLAY_SERVICE_ACCOUNT_JSON']` in Fastlane `Fastfile` lanes to guard against
trailing whitespace or newlines injected by CI secret bindings.
