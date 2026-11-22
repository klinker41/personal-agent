---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-11-22T07:20:35.555393+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Repository at `/workspace/antigravity-plugin` (configured in
`~/.gemini/config/plugins.json`, tracking `main` from
`git@github.com:klinker41/antigravity-plugin.git`). Houses rules, skills,
lifecycle hooks (`hooks.json`), sidecars (`sidecar.json`), MCP configuration
(`mcp_config.json` in plugin root, not global directory), and
`vendor/agent-skills`.

## Development Rules & Invariants
- **Container & Services:** Protect container runtime: never kill
  `auth-proxy.js`, infra ports (4400, 4402, 4405, 7681, 8080), or run blanket
  `pkill bun` (`rules/docker-container-environment.md`). Web apps require Bun,
  Hono, minimal dependencies (`rules/web-app-architecture.md`), and port 4401
  routed to `https://prototype.klinker-cabin.computer`
  (`rules/web-service-port.md`).
- **Coding & Models:** Use kebab-case naming, 80-column Markdown wrapping,
  minimal diffs (`rules/simplify-changes.md`), and README updates
  (`rules/readme-updates.md`). Subagents use `Model: "flash"` for coding/tests
  (`rules/coding-subagent-model.md`) and `Model: "pro"` for self-reviews.
- **Git Quality Gates:** Tests must pass non-verbosely (`rules/tests.md`,
  `rules/non-verbose-tests.md`), commits must be secret-free
  (`rules/no-secrets-in-commits.md`) and vetted by `self-review-commit`
  (`rules/self-review-before-commit.md`). Never `git push` without explicit
  approval (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Store & Injection:** Progressive disclosure hierarchy under
  `$MEMORY_DIRECTORY` (`profile.md`, `index.md`, `projects/`, `knowledge/`, and
  chronicles `daily/`, `monthly/`, `yearly/`) via `lookup-memory` and
  `save-memory`. First turn injects memory on `initialNumSteps == 0` and
  `invocationNum == 1` (`hooks/inject_memory.py`).
- **Nightly Pipeline:** Automated sequence: 00:00 Dreamer session extraction
  (watermarks in `state.json`, topic resolution via `get_existing_topics`,
  subagent/rule filtering, ephemeral session purge); 00:30 Compactor chronicle
  consolidation and semantic refinement (`compactor.py`); 01:00 Git sync with
  secret scrubbing (`git_sync.py`).

## Sidecars & Integrations
- **Agent API Bridge (`utils/memory_utils.py`):** Subprocess runner with auth
  retries resolving projects in `~/.gemini/config/projects/` by name, UUID,
  or path (defaults to `personal-agent`, UUID
  `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or `$ANTIGRAVITY_PROJECT_ID`). Cleans
  env (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) and extracts `ANTIGRAVITY_LS_ADDRESS` and CSRF
  tokens (`window.__APP_CONFIG__.csrfToken`) from `cli.log` and runtime state.
- **Scheduled Tasks & Slack Daemon (`sidecar.json`):** Recurring cron jobs run
  daily Gemini `model-updater` (15:00 UTC) and weekly `vendor/agent-skills`
  `submodule-updater` (Mondays 15:00 UTC). Slack daemon (`sidecars/slack-chat/`,
  prepped via `prep-slack-chat-sidecar`) bridges Socket Mode threads to
  `agentapi` (`agy agentapi new-conversation --model=<tier>`) using tiers in
  `sidecar.json` (e.g., `model: 'pro'`) with backfill; outbound alerts use
  `notify-the-user`.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.

- Continuous Python sidecar workers (`slack_daemon.py` and `memory_daemon.py`)
managed by `sidecar-manager.js` (`restart_policy: "always"`) can be reloaded
with updated shared library code (`utils/memory_utils.py`) by terminating their
individual processes so `sidecar-manager.js` respawns them, without needing to
restart or redeploy `antigravity-docker`.

- The `growatt-time-sync` scheduled sidecar
(`sidecars/growatt-time-sync/sidecar.json`) runs weekly (`0 6 * * 0`) using
`"model": "flash"`.

- Added the `instagram-reels-downloader` scheduled sidecar
(`sidecars/instagram-reels-downloader/`, running on cron `0 11,19 * * *` with up
to 30 minutes of random jitter on `:00` scheduled runs) to poll
`https://www.instagram.com/liveinneon/reels/` via `yt-dlp`, store videos and
`.yt-dlp-archive.txt` in `/reels`, read session cookies from
`/instagram/cookies.txt`, and upload newly downloaded `.mp4` files directly to
the user's Slack DM using `$SLACK_BOT_TOKEN` and `files_upload_v2`.

- Sidecar model tier allocation: `slack-chat` (with `slack_daemon.py` capping
Decisions API `model_tier` routing so conversations never escalate above
`flash`), `aibox-updater`, and `jellyfin-log-checker` (`max_log_files: 2`) use
`flash`; routine maintenance sidecars (`podcast-organizer`,
`sonarr-import-cleaner`, `pokemon-episode-namer`, `growatt-time-sync`,
`dangling-container-cleaner`, and the consolidated Radarr/Sonarr quality and
series monitoring checker) use `flash_lite`; `sidecar-auditor` runs Fridays on
`pro` without a Python pre-filter to audit `brain/` transcripts, daemon logs,
MCP calls, `slack-chat`, and `memory-daemon`.
- Removed the daily `service-auditor` (`Workspace Service Log Auditor`) sidecar
due to low signal and consolidated the separate Radarr and Sonarr monitored
quality/series checkers into a single sidecar.
- `utils/sidecar_prefilter.py` runs `dangling-container-cleaner`
(`komodo_prune_containers` on `0 5,17 * * *`) deterministically in Python with
zero agent turns during normal operation.

- Consolidated `pro-model-updater` and `tts-model-updater` into a single
`model-updater` sidecar (`sidecars/model-updater/sidecar.json`) backed by a thin
fetch-only `skills/model-updater/scripts/fetch_models.py` script rather than
hardcoded Python ranking tables.
- `skills/model-updater/SKILL.md` is the single source of truth for tier
preferences (`pro`: `gemini-4-carbon-high`, `flash`: `gemini-4-neon-medium`,
`flashLite`: `gemini-4-hydrogen-medium`) and known codename progressions; when
unrecognized codenames appear in the catalog, `model-updater` leaves
repositories untouched and sends a Slack alert.

- Added `utils/github_app.py` to lazily bootstrap the `gh` CLI into
`~/.local/bin/gh`, sign RS256 GitHub App JWTs via `openssl` + Python stdlib with
zero pip dependencies, and mint 1-hour installation tokens (`python3
utils/github_app.py token | ensure-gh | repos`).
- Added the `github-watcher` sidecar (`sidecars/github-watcher/`), scheduled
every 10 minutes (`*/10 * * * *`) on `flash`, which polls GitHub issues and PRs
scoped to local `/workspace/*` repos, seeds initial `state.json` on first run,
filters out `jklinker-antigravity-bot[bot]` events to prevent loops, and uses
disposable `git worktree` checkouts to investigate issues, open well-scoped fix
PRs, and review/update owner-authored PRs.
- `github-watcher` leaves spawned `agentapi` conversations unarchived and sends
Slack notifications via `notify-the-user` containing
`https://antigravity.klinker-cabin.computer/c/<conversation_id>`, allowing
`slack-chat` to automatically map `thread_ts -> conversation_id` and route Slack
thread replies back into the same session.
- Updated `rules/commit-co-author.md` and sidecar co-author trailers to credit
the GitHub App bot identity `jklinker-antigravity-bot[bot]
<359595847+jklinker-antigravity-bot[bot]@users.noreply.github.com>`.

- `sidecars/jellyfin-upstream-sync/sidecar.json` runs every 12 hours
(`0 */12 * * *`) on `flash` via `utils/sidecar_prefilter.py`
(`jellyfin-upstream-sync` mode) to merge new `MakD/AFinity` commits into
`jellyfin-client` without spending LLM tokens when 0 upstream commits exist, and
is registered in `sidecar-auditor`.

- In `sidecars/slack-chat/slack_daemon.py`, Jenkins build notifications are
matched in Stage 0 (`ACTIONABLE_ALERT_PATTERNS` and `RECOVERY_ALERT_PATTERNS`),
keyed per job as `jenkins-<slug>` (`_is_jenkins_incident_key()`), isolated to
exact-only correlation (bypassing fuzzy/burst homelab alert merging), and routed
to matching `/workspace/<project>` targets via `_resolve_alert_target`.
- The `slack-chat` sidecar persists active Slack thread correlation and
`incident_key` state at
`/home/developer/.gemini/antigravity/sidecar_data/antigravity-plugin/slack-chat/data/state.json`.

- Commit Co-authored-by trailers across plugin rules
(rules/commit-co-author.md), integrity tests, and sidecar prompts use
Antigravity as the display name paired with the GitHub App noreply address:
Co-authored-by: Antigravity
<359595847+jklinker-antigravity-bot[bot]@users.noreply.github.com>.

- Rule markdown files under rules/ (such as rules/simplify-changes.md) are
validated by content assertions in tests/test_plugin.py (run via python3 -m
unittest discover -s tests), so rule edits require corresponding test updates.

- Enforces git commit attribution via the always-on rules/commit-co-author.md
rule (validated in tests/test_plugin.py), appending a Co-authored-by trailer for
Antigravity to every commit message.

- Includes the scheduled `jellyfin-dependency-updater` sidecar
(`jellyfin_deps_checker.py`) to routinely check and update Gradle, version
catalog, and Android tooling dependencies in `/workspace/jellyfin-client`,
migrating breaking API changes instead of deferring updates and notifying the
user when code changes are made.
