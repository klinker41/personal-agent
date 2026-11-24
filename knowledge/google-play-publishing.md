---
topic: google-play-publishing
category: knowledge
tags: [knowledge, google-play-publishing]
updated_at: 2026-11-24T07:17:00.920648+00:00
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

- Fastlane Google Play uploads (`google_play_track_version_codes` and
`upload_to_play_store`) require enabling `androidpublisher.googleapis.com` on
the service account's GCP project, uploading the initial `.aab` manually once in
Play Console, and granting the service account View app information and Release
apps to testing tracks permissions (saving a minor store listing edit forces
Play Console permission propagation in minutes instead of up to 24 hours).
- When an app is still in Draft state in Google Play Console, the Play Developer
API rejects `release_status = completed` with `Only releases with status draft
may be created on draft app`; automated uploads must pass `release_status =
draft` until an initial release is submitted to exit Draft state.

- Google Play API error `The caller does not have permission` occurs when a
service account only has account-level permissions without explicit per-app
access in Play Console -> Users and permissions -> App permissions (requires
View app information, Release apps to testing tracks, and Manage testing
tracks), when no initial `.aab` has been uploaded manually, or when the Google
Play Android Developer API is disabled in the owning GCP project.
- Google Play API error `Only releases with status draft may be created on draft
app` occurs while an app has Draft status in Play Console, even if a manual
internal test release is live. Automated `completed` API rollouts to
internal/closed tracks require completing Dashboard app setup tasks and
submitting the internal track for a one-time review in Publishing overview
(which exits Draft status while keeping Production inactive); until approved, CI
must pass release status `draft`.
- Google Play rejects uploading the same `.aab` version code more than once
across tracks; to release one build to multiple tracks (e.g., `internal` and
`alpha`) in Fastlane, upload to the primary track first and promote that release
to each additional track.
