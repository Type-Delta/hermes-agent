# Type-Delta Hermes Agent Fork

This fork carries the local Discord identity controls and media-development
support used by the Type-Delta Hermes installation.

## Divergence Log

This is a current-state record only. Each entry describes a surviving
difference between `HEAD` and the latest shared base, determined with
`git merge-base HEAD upstream/main` (currently
`62e8126c6929d1a5d9fd650317214ef240767d4e`). A feature adopted from upstream
is not a divergence merely because it was involved in a merge.

Keep stable IDs when updating this section. When upstream absorbs a difference,
remove or rewrite the entry rather than preserving chronology here. Update its
behavior, implementation evidence, and validation when the surviving difference
changes.

### HD001 — Discord gateway intents and stable model-facing identity

Operators can set `discord.gateway_intents` to an exact Discord Gateway Intents
bitfield. When it is absent, Hermes retains upstream's least-privilege automatic
intent selection.

Discord turns identify human senders to the model by their unique account
username, with immutable numeric user ID as the fallback. Display names and
guild nicknames are never used for model-facing identity. The rule applies to
ordinary messages, slash-command turns, thread starters, and channel-history
backfill.

**Implementation evidence:** `hermes_cli/config_defaults.py`,
`plugins/platforms/discord/adapter.py`, and
`tests/gateway/test_discord_{connect,free_response,slash_commands}.py`.

**Recorded validation:** focused Discord connect, free-response, slash-command,
context-recovery, and adapter tests; Ruff checks for the affected Python files.

**Last updated:** 2026-08-29

### HD002 — Configurable KittenTTS backend and browser development support

KittenTTS model construction accepts the configured `tts.kittentts.backend`,
defaulting to `cpu`, while retaining the shared model cache. The root JavaScript
development dependencies also include Playwright so the repository can run
local browser automation without an ad hoc install.

**Implementation evidence:** `tools/tts_tool.py`,
`tests/tools/test_tts_kittentts.py`, `package.json`, and `package-lock.json`.

**Recorded validation:** Python byte-compilation, JSON and lockfile parsing,
dependency resolution with lifecycle scripts disabled, and the focused TTS
test slice.

**Last updated:** 2026-08-29
