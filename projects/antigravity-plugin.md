---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-11-28T07:37:02.996095+00:00
compacted_at: 2026-11-28T07:37:02.996095+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Repository at `/workspace/antigravity-plugin` (configured in
`~/.gemini/config/plugins.json`, tracking `main` from
`git@github.com:klinker41/antigravity-plugin.git`). Houses rules (`rules/`),
skills (`skills/`), lifecycle hooks (`hooks.json`), sidecars
(`sidecars/*/sidecar.json`), MCP config (`mcp_config.json` in plugin root, not
global directory), and `vendor/agent-skills`.

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
- **Git Gates & Attribution:** Tests must pass non-verbosely (`rules/tests.md`,
  `rules/non-verbose-tests.md`). Commits must be secret-free
  (`rules/no-secrets-in-commits.md`), vetted by `self-review-commit`
  (`rules/self-review-before-commit.md`), and never pushed without explicit
  approval (`rules/git-push.md`). Always-on `rules/commit-co-author.md` appends
  the trailer `Co-authored-by: Antigravity
  <359595847+jklinker-antigravity-bot[bot]@users.noreply.github.com>` across
  rules, integrity tests, and sidecar prompts.
- **Rule Integrity Tests:** Run `python3 -m unittest discover -s tests`.
  `tests/test_plugin.py` asserts exact contents of `rules/` files (such as
  `rules/simplify-changes.md` and `rules/commit-co-author.md`), so rule edits
  require corresponding test updates.

## Memory System (`sidecars/memory-daemon`)
- **Store & Injection:** Progressive disclosure hierarchy under
  `$MEMORY_DIRECTORY` (`profile.md`, `index.md`, `projects/`, `knowledge/`, and
  chronicles `daily/`, `monthly/`, `yearly/`) via `lookup-memory` and
  `save-memory`. First turn injects memory on `initialNumSteps == 0` and
  `invocationNum == 1` (`hooks/inject_memory.py`).
- **Nightly Pipeline:** Automated consolidation for `personal-agent`:
  00:00 Dreamer (`dreamer.py`: watermarks in `state.json`, topic resolution via
  `get_existing_topics`, subagent/rule filtering including
  `is_subagent_conversation` detection of `/swarm` workers, ephemeral session
  purge); 00:30 Compactor (`compactor.py`: chronicle consolidation and semantic
  refinement); 01:00 Git sync with secret scrubbing (`git_sync.py`).
- **Agent API Bridge (`utils/memory_utils.py`):** Subprocess runner with auth
  retries and fixed `ask_llm` timeout handling for long `pro`-model compaction
  and dreaming runs. Resolves projects in `~/.gemini/config/projects/` by name,
  UUID, or path (defaults to `personal-agent`, UUID
  `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or `$ANTIGRAVITY_PROJECT_ID`). Cleans
  env (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) and extracts `ANTIGRAVITY_LS_ADDRESS` and CSRF
  tokens (`window.__APP_CONFIG__.csrfToken`) from `cli.log` and runtime state.

## Sidecar Architecture & Model Tiers
- **Runtime & Storage:** Sidecars live in `sidecars/*/sidecar.json` with data
  and zero-agent pre-filter logs under
  `/home/developer/.gemini/antigravity/sidecar_data/antigravity-plugin/`
  (`*/data/`, `*/logs/`). Completed scheduled runs archive conversations
  (`archived:true`). Continuous Python workers (`slack_daemon.py`,
  `memory_daemon.py`) managed by `sidecar-manager.js` (`restart_policy:
  "always"`) reload updated `utils/memory_utils.py` code when terminated
  individually, without restarting `antigravity-docker`.
- **Model Tier Allocation:** `pro`: `sidecar-auditor` (Fridays, no Python
  pre-filter; audits `brain/` transcripts, daemon logs, MCP calls,
  `slack-chat`, `memory-daemon`, and `jellyfin-upstream-sync`). `flash`:
  `slack-chat`, `github-watcher`, `aibox-updater`, `jellyfin-log-checker`
  (`max_log_files: 2`), and `jellyfin-upstream-sync`. `flash_lite`: routine
  maintenance (`podcast-organizer`, `sonarr-import-cleaner`,
  `pokemon-episode-namer`, `growatt-time-sync`, `dangling-container-cleaner`,
  and the consolidated Radarr/Sonarr quality and series monitoring checker).
- **Retirements:** Removed daily `service-auditor` (`Workspace Service Log
  Auditor`) due to low signal and merged the separate Radarr and Sonarr
  monitoring checkers into one sidecar.

## Daemons & Event Integrations
- **Slack Daemon (`sidecars/slack-chat/slack_daemon.py`):** Prepped via
  `prep-slack-chat-sidecar`; bridges Socket Mode threads to `agentapi`
  (`agy agentapi new-conversation --model=<tier>`) with backfill, capping
  Decisions API `model_tier` routing at `flash`. Persists thread correlation
  (`thread_ts -> conversation_id`) and `incident_key` state in
  `slack-chat/data/state.json` and ambient streams in `ambient/*.jsonl`.
  Handles flapping alert correlation and `media-generator` webhook triage.
  Stage 0 (`ACTIONABLE_ALERT_PATTERNS`, `RECOVERY_ALERT_PATTERNS`) matches
  Jenkins notifications keyed as `jenkins-<slug>` (`_is_jenkins_incident_key`),
  using exact-only correlation (no fuzzy/burst homelab merging) and routing to
  `/workspace/<project>` via `_resolve_alert_target`. Outbound alerts use
  `notify-the-user`.
- **GitHub Watcher (`sidecars/github-watcher/`, `utils/github_app.py`):**
  `utils/github_app.py` lazily bootstraps `gh` into `~/.local/bin/gh`, signs
  RS256 GitHub App JWTs via `openssl` + Python stdlib (zero pip deps), and
  mints 1-hour installation tokens (`python3 utils/github_app.py token |
  ensure-gh | repos`). `github-watcher` runs every 10 minutes (`*/10 * * * *`)
  on `flash`, polling GitHub issues and PRs for local `/workspace/*` repos,
  seeding `state.json` on first run, filtering `jklinker-antigravity-bot[bot]`
  events to prevent loops, and using disposable `git worktree` checkouts to
  investigate issues, open scoped fix PRs, and review/update owner-authored
  PRs. Leaves spawned `agentapi` conversations unarchived and sends Slack
  alerts via `notify-the-user` containing
  `https://antigravity.klinker-cabin.computer/c/<conversation_id>` so
  `slack-chat` maps `thread_ts -> conversation_id`.

## Scheduled Sidecars & External Skills
- **Updaters:** Consolidated `pro-model-updater` and `tts-model-updater` into
  daily `model-updater` (15:00 UTC, `sidecars/model-updater/sidecar.json`)
  backed by thin fetch-only `skills/model-updater/scripts/fetch_models.py` (no
  hardcoded Python ranking tables). `skills/model-updater/SKILL.md` is the
  single source of truth for tier preferences (`pro`: `gemini-4-neon-xhigh`,
  `flash`: `gemini-4-neon-medium`, `flashLite`: `gemini-4-hydrogen-medium`) and
  known codename progressions; unrecognized codenames leave repos untouched and
  send a Slack alert. Weekly `submodule-updater` updates `vendor/agent-skills`
  (Mondays 15:00 UTC).
- **Homelab & Maintenance:** `dangling-container-cleaner` runs
  `komodo_prune_containers` (`0 5,17 * * *`) deterministically in
  `utils/sidecar_prefilter.py` with zero agent turns during normal operation.
  `growatt-time-sync` (`sidecars/growatt-time-sync/sidecar.json`) runs weekly
  (`0 6 * * 0`) on `flash_lite`. `jellyfin-upstream-sync`
  (`sidecars/jellyfin-upstream-sync/sidecar.json`, `0 */12 * * *`, `flash`)
  uses `utils/sidecar_prefilter.py` (`jellyfin-upstream-sync` mode) to merge
  `MakD/AFinity` commits into `jellyfin-client` with zero LLM tokens when no
  upstream commits exist. `jellyfin-dependency-updater`
  (`jellyfin_deps_checker.py`) updates Gradle, version catalog, and Android
  tooling in `/workspace/jellyfin-client`, migrating breaking API changes
  instead of deferring and notifying the user when code changes are made.
- **Media & Animation Pipelines:** `instagram-reels-downloader`
  (`sidecars/instagram-reels-downloader/`, `0 11,19 * * *` with up to 30m
  random jitter on `:00` runs) polls
  `https://www.instagram.com/liveinneon/reels/` via `yt-dlp`, saves videos and
  `.yt-dlp-archive.txt` in `/reels`, reads cookies from
  `/instagram/cookies.txt`, and uploads new `.mp4` files to the user's Slack DM
  via `$SLACK_BOT_TOKEN` and `files_upload_v2`. Renamed
  `sidecars/movie-chapter-generator/sidecar.json` (verified in
  `tests/test_movie_chapter_generator.py`) is restricted to Harry Potter movies
  only (cel-animation pipeline, avoiding other `animation-generator` projects).
  Added an initially disabled Daily DCC Movie Generator sidecar for
  `animation-generator/projects/dcc-floor-1/parts.json` (resolving the next
  incomplete part or `NOOP_ARCHIVE_ONLY`); `tests/test_dcc_movie_generator.py`
  derives the expected target dynamically from live `parts.json` rather than
  hardcoding `target_part == 1`.
- **External AI Skills:** `ollama-chat` queries Gemma models at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
