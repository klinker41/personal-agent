---
topic: github-app-auth
category: knowledge
tags: [knowledge, github-app-auth]
updated_at: 2026-11-22T07:15:27.440172+00:00
confidence: 0.95
---

# Knowledge: Github-App-Auth

- GitHub App RS256 JWTs can be generated with zero external Python packages by
signing the header/payload with `openssl dgst -sha256 -sign` and exchanging the
JWT at `/app/installations/<id>/access_tokens` for a 1-hour `GH_TOKEN`, paired
with `gh auth setup-git` for git push authentication.
- Authenticating automation via a GitHub App (`<app>[bot]`) instead of a
personal token enables submitting formal PR reviews (approve or request changes)
on the repository owner's own PRs.
- To prevent infinite PR review loops when a bot pushes fix commits, git
commands must explicitly set `-c user.name='<app>[bot]'` and `-c
user.email='<bot-user-id>+<app>[bot]@users.noreply.github.com'` so GitHub
resolves `committer.login` to `<app>[bot]`.

- GitHub disables Issues by default on forked repositories
(`hasIssuesEnabled: false`); a GitHub App installation token with
`issues: write` cannot file issues on a fork until Issues are enabled in
repository settings (or the app has `administration: write`).
