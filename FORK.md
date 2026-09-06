# Type-Delta Hermes Agent Fork

This fork carries the local Discord configuration and identity controls plus
media-development support used by the Type-Delta Hermes installation.

## Divergence Log

This is a current-state record only. Each entry describes a surviving
difference between `HEAD` and the latest shared base, determined with
`git merge-base HEAD upstream/main` (currently
`01ae7a5668ce0fa2efca524a4567cacdd0786c95`). A feature adopted from upstream
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
