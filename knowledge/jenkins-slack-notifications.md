---
topic: jenkins-slack-notifications
category: knowledge
tags: [knowledge, jenkins-slack-notifications]
updated_at: 2026-11-22T07:11:38.433985+00:00
confidence: 0.95
---

# Knowledge: Jenkins-Slack-Notifications

- The Jenkins Slack Notification plugin authenticates via a Slack App Bot User
OAuth Token (xoxb-...) stored as a Jenkins 'Secret text' credential, with bot
scopes chat:write, chat:write.public, channels:read, groups:read, files:write,
reactions:write, users:read, and users:read.email, and requires inviting the bot
to target private channels.
- Modern Slack App Incoming Webhooks ignore username and icon_url JSON payload
overrides (tying sender identity to the Slack App itself); for Jenkins instances
with UI-configured jobs lacking Jenkinsfiles, global webhook listeners across
all builds can also be registered via hudson.model.listeners.RunListener in
$JENKINS_HOME/init.groovy.d/.

- Incident correlation for Jenkins Slack alerts must derive per-job keys from
URL-decoded `/job/<segment>` path components (`urllib.parse.unquote`) or
notification job titles rather than falling back to the shared Jenkins hostname,
which otherwise merges unrelated job failures into an existing unrecovered alert
thread.
