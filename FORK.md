# Type-Delta Hermes Agent Fork

This fork carries the local Discord configuration and identity controls plus
media-development support used by the Type-Delta Hermes installation.

## Divergence Log

This is a current-state record only. Each entry describes a surviving
difference between `HEAD` and the latest shared base, determined with
`git merge-base HEAD upstream/main` (currently
`d76856cc6971b6e0e1903b5369498bcc4bb83a60`). A feature adopted from upstream
is not a divergence merely because it was involved in a merge.

Keep stable IDs when updating this section. When upstream absorbs a difference,
remove or rewrite the entry rather than preserving chronology here. Update its
behavior, implementation evidence, and validation when the surviving difference
changes.

### HD001 — Explicit Discord gateway intents

Operators can set `discord.gateway_intents` to an exact Discord Gateway Intents
bitfield. When it is absent, Hermes retains upstream's least-privilege automatic
intent selection.

**Implementation evidence:** `hermes_cli/config_defaults.py`,
`plugins/platforms/discord/adapter.py`, and
`tests/gateway/test_discord_connect.py`.

**Recorded validation:** Discord connect and adjacent gateway/config tests via
`scripts/run_tests.sh`; real CLI set/get persisted the integer intent value in
a temporary `HERMES_HOME`. Raw YAML parsing covered explicit zero, configured
values, missing values, and invalid values.

**Last updated:** 2026-09-06

### HD002 — Stable model-facing Discord identity

Ordinary Discord messages, slash-command turns, thread starters, and
channel-history backfill identify human senders to the model by their unique
account username, with immutable numeric user ID as the fallback. These paths
do not use display names or guild nicknames for sender identity. Voice
participant labels and platform lifecycle-hook metadata retain upstream behavior.

**Implementation evidence:** `plugins/platforms/discord/adapter.py` and
`tests/gateway/test_discord_{free_response,slash_commands}.py`.

**Recorded validation:** Discord free-response, slash-command, and adjacent
identity/authorization tests via `scripts/run_tests.sh`, including history labels
and account-name fallback coverage.

**Last updated:** 2026-09-06

### HD003 — Configurable KittenTTS backend and browser development support

KittenTTS model construction accepts the configured `tts.kittentts.backend`,
defaulting to `cpu`, through upstream's local-engine loader used for both speech
generation and model warm-up. Cache keys include both model and backend, so
switching backends does not reuse a model loaded for the previous backend.
The root JavaScript development dependencies also include Playwright so the
repository can run local browser automation without an ad hoc install.

**Implementation evidence:** `tools/tts_tool_local.py`,
`tests/tools/test_tts_kittentts.py`, `package.json`, and `package-lock.json`.

**Recorded validation:** KittenTTS, model-cache LRU, lifecycle lease, streaming,
and profile-output tests via `scripts/run_tests.sh`. The backend-cache invariant
checks reuse for the same backend and isolation between CPU and CUDA selections.
`npm ci --dry-run --ignore-scripts --no-audit --no-fund` resolved the lockfile.
Model construction is mocked in these tests; live TTS inference was not run.

**Last updated:** 2026-09-06

### HD004 — Discord bot-admission policy in YAML

Operators can set `discord.allow_bots` to `none`, `mentions`, or `all` in
`config.yaml`, with `DISCORD_ALLOW_BOTS` retaining precedence when both are
present. The setting is carried in each adapter's profile-scoped configuration
so multiplexed profiles do not leak bot-admission policy through process-global
environment state.

**Implementation evidence:** `plugins/platforms/discord/adapter.py`,
`gateway/authz_mixin.py`, `tests/gateway/test_discord_bot_auth_bypass.py`,
`tests/plugins/platforms/test_discord_gate_isolation.py`, and
`website/docs/user-guide/messaging/discord.md`.

**Recorded validation:** Discord bot-authorization, bot-filter, and profile
gate-isolation tests via `scripts/run_tests.sh`. Checks cover environment `none`
overriding permissive YAML and reject YAML bot bypass on unsupported platforms.
Real CLI set/get persisted YAML bot policy in a temporary `HERMES_HOME`.

**Last updated:** 2026-09-06

### HD005 — Release-triggered installation checks

The fork removes the twice-daily schedule from the Install & Update E2E
workflow. Release-tag pushes and manual dispatch still run the existing jobs,
using upstream's current dispatch inputs (`route`, `tag-count` defaulting to
`3`, and `install-ref`).

**Implementation evidence:** `.github/workflows/install-e2e.yml`.

**Recorded validation:** `git diff upstream/main` on the workflow shows only the
schedule trigger and its two comment lines removed; other triggers, dispatch
inputs, and all job definitions match upstream.

**Last updated:** 2026-09-12

## Repository governance

`AGENTS.md` retains this fork's remote, divergence-log, consent, and completion
requirements. This document records the code differences above and the history
below. These repository instructions and records do not add runtime behavior.

## Merge History

This is an append-only historical decision record. It provides context for
integrations but never, by itself, establishes an ongoing fork divergence; use
the current Divergence Log for that determination.

Update the `base` tag after each verified merge to the integrated commit. That
tag records the fork's completed integration; `git merge-base HEAD upstream/main`
identifies the shared upstream ancestor.

### 2026-09-06 — Merge upstream/main into main

**Merge commit:** the upstream integration commit containing this entry,
identified by the parents below.
**Parents:** `0be052de361cb2f688ed4f81dcd452c22cde2204` (fork) and
`01ae7a5668ce0fa2efca524a4567cacdd0786c95` (upstream/main).

- Integrate upstream into the existing `sync-upstream` worktree branch first,
  then merge that branch back to `main`, as requested by the user.
- Adopt upstream's module decomposition and current implementation structure.
  Keep all four fork behavior IDs because the target has not absorbed them.
- Retain HD001's exact Discord intent override alongside upstream's automatic
  intent defaults, and HD002's account-based identity on messages, commands,
  thread starters, and history backfill.
- Retain HD004's profile-scoped YAML bot-admission behavior within upstream's
  authorization structure. An explicit environment value of `none` also wins
  over permissive YAML, matching the documented environment override contract.
  Review caught and fixed an initial merge regression that allowed the YAML
  fallback on unsupported platforms; the upstream platform boundary is retained.
- Move HD003's KittenTTS backend argument to the local-engine loader so upstream's
  model warm-up and generation paths share the same selection. Key the model
  cache by backend as well as model name to keep backend switches effective.
  Retain Playwright as a development dependency.
- Accept upstream additions including built-in `/plan`, contextual `/btw`, `/bg`,
  default tool deferral, local models, provider and browser expansion, Group Chat
  recovery, remote sandbox media delivery, and security fixes.
- Preserve the requested append-only Merge History format and replace the user's
  example entry with this integration record.

**Validation:** 220 tests passed across 17 focused/adjacent files, with 2 skips
and no failures or flaky retries. After adding explicit environment-precedence
coverage, the two affected files passed again with 29 tests. Real isolated
CLI/config and adapter YAML parsing checks passed; npm dependency dry-run passed.
Python compilation, focused Ruff checks, and the compatibility-pointer scanner
also passed. Combined distinct targeted coverage is 222 passed and 2 skipped.
The fork-only diff passes `git diff --check upstream/main`. The complete upstream
suite, live Discord connection, and real TTS inference were not run.

### 2026-09-12 — Merge upstream/main into main

**Merge commit:** `015e50d944321219cf37a9e8c6b865b9c0ea5c49`.
**Parents:** `62fbe865543624f7b454a4472a1c6f318ac953fc` (fork) and
`d76856cc6971b6e0e1903b5369498bcc4bb83a60` (upstream/main).

- Merge upstream directly into `main` (1788 upstream commits, about 2800
  files) with the owner's explicit consent recorded on 2026-09-12. No rebase.
- The only textual conflict was the Install & Update E2E workflow: upstream
  changed the `tag-count` default to `3` and added an `install-ref` input in
  the same region where HD005 removes the schedule trigger. Take upstream's
  inputs and keep HD005's schedule removal. This is not an incompatible
  product choice; both sides survive.
- Git auto-merged HD001, HD002, and HD004 in the Discord adapter, config
  defaults, and authorization mixin, and HD003 in the local TTS loader and
  package files. Upstream has not absorbed any of the five behavior IDs, so
  all entries remain.
- Accept upstream additions including the multiplexed gateway migration,
  shared-listener ingress, cron catch-up after restart gaps, gateway turn-lease
  and failed-turn fixes, Codex pool inheritance, desktop onboarding, and the
  version bump to 0.21.2. Live gateway was not restarted; the running process
  still executes the pre-merge code until its next scheduled restart.

**Validation:** The six fork-specific files (Discord connect, free-response,
slash-command, bot-auth bypass, gate isolation, KittenTTS) passed with 95 tests
and no failures via `scripts/run_tests.sh`. Broader subsets under `tests/gateway/`,
`tests/plugins/`, `tests/tools/`, and `tests/hermes_cli/` ran 28,506 passed, 78
failed, 291 skipped; `tests/agent/` ran 6,887 passed, 16 failed, 32 skipped.
Every failing file was rerun against a pristine `upstream/main` worktree with the
same virtualenv: all but three failed identically there (optional packages such
as `daytona`, `fal-client`, `modal`, `parallel-web`, `hindsight-client-api`, and
`anthropic` are not installed, and lazy installs are disabled). The remaining
three (`test_config_backups.py`, `test_delegate_timeout_cleanup.py`,
`test_compression_stall_fallback_78981.py`) are timing-dependent in code that is
byte-identical to upstream; the latter two passed on rerun, and the first is a
`filecmp` mtime-granularity race reproduced outside pytest. No failure involves a
fork-touched file. Python compilation of the fork-touched modules, `git diff
--check upstream/main`, and `npm ci --dry-run --ignore-scripts` passed. The
complete suite, live Discord connection, real TTS inference, and a gateway
restart were not run.
