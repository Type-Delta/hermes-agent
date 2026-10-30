# Type-Delta Hermes Agent Fork

<!-- In-synced fork: maintained with the `syncfork` skill. -->

This fork carries explicit Discord gateway intents, username-based Discord
sender identity, and GPU-backed local voice (KittenTTS and faster-whisper) for
the Type-Delta Hermes installation.

## Divergence Log

Each behavior where this fork differs from upstream has one entry here. If you change code in this repository, check if this log needs an update.

The rules:

- Update the entry that owns the behavior you changed, or add an entry with the next unused ID. One entry covers one behavior, not one commit.
- Describe the current behavior, why the fork needs it, and how it composes with upstream's related behavior. When the behavior changes, rewrite the entry. Do not append a changelog; git and the Merge History hold history.
- Under **Implementation evidence**, list the key source files and tests. Under **Recorded validation**, list the checks that proved the current behavior.
- Set **Last updated** to today's date on every entry you touch.
- Never reuse an ID. Retire an entry only with the user's explicit approval: delete it and name the retired ID in the next Merge History entry.
- When an ID conflicts from merges, renumber the IDs as needed.
- A change that does not alter how the fork differs from upstream (formatting, or an edit to this file) needs no entry.

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
do not use display names or guild nicknames for sender identity. Voice-channel
turns label the speaker the same way (account username, then the join-time
name, then the cached user's username), so spoken and typed turns render the
same pinned session context. Platform lifecycle-hook metadata, including message
edit/delete events, retains upstream behavior.

**Implementation evidence:** `plugins/platforms/discord/adapter.py`,
`gateway/run_voice.py`, and
`tests/gateway/test_discord_{free_response,slash_commands,turn_prompt_parity}.py`.

**Recorded validation:** Discord free-response, slash-command, and adjacent
identity/authorization tests via `scripts/run_tests.sh`, including history labels
and account-name fallback coverage.

**Last updated:** 2026-10-25

### HD003 — GPU local voice (KittenTTS and faster-whisper) and browser development support

`tts.kittentts.backend` selects `cpu` (default) or `cuda`. With `cuda`, the
loaded KittenTTS model's ONNX session is rebuilt on `CUDAExecutionProvider`,
and loading fails loudly instead of silently running on CPU. Model-cache keys
include the backend. Local faster-whisper with a non-CPU device preloads the
same pip-installed CUDA libraries, so `stt.local.device: cuda` works without
system CUDA.

Packaging: on Linux x86_64, `onnxruntime-gpu==1.25.1` replaces `onnxruntime`
in the `kittentts`, `piper`, `voice`, and `stt-whisper` extras, and a `[tool.uv]`
override drops the CPU wheel there (both install the same `onnxruntime`
package). PM's `uv pip check` reader accepts the resulting "requires
`onnxruntime`, but it's not installed" line only when the lock carries
`onnxruntime-gpu`. The opt-in `onnx-cuda` extra pins CUDA 12.4 / cuDNN 9.1
libraries, which still run on Pascal GPUs and R550 drivers; newer cuDNN 9.x
fails at execution there. torch, which misaki[en] imports, comes from the
PyTorch CPU index on Linux x86_64, because the default wheel installs a CUDA 13
cuDNN over the same `nvidia/cudnn` directory. Root JavaScript development
dependencies also include Playwright.

**Implementation evidence:** `tools/tts_tool_local.py`,
`tools/transcription_local.py`, `pm/pip_check.py`, `pm/extras.py`,
`pyproject.toml`, `uv.lock`, `tests/tools/test_tts_kittentts.py`,
`tests/pm/test_pip_check.py`, `package.json`, and `package-lock.json`.

**Recorded validation:** KittenTTS, PM pip-check/extras, transcription, and
project-metadata tests via `scripts/run_tests.sh` (150 passed). Live checks on a
GTX 1060 (driver 550, CUDA 12.4) went through Hermes's real `text_to_speech_tool`
and `transcribe_audio`. KittenTTS nano-fp32 ran on `CUDAExecutionProvider` at
about 0.35 CPU-seconds per warm sentence. faster-whisper medium int8 on CUDA
transcribed the result correctly. The int8 KittenTTS models (mini, micro) run
their `ConvInteger` nodes on the CPU under any provider, so only the fp32 model
offloads.

**Last updated:** 2026-10-30

## Sync Checklist

Project-specific checks every upstream sync runs, in addition to the full test suite, lint, typecheck, builds, and a smoke test of upstream and fork features.

- Take upstream's `pyproject.toml` and `uv.lock`, reapply HD003's
  `pyproject.toml` edits, and regenerate the lock with `hermes pm lock`.
  Confirm the lock carries `onnxruntime-gpu` and the PyTorch CPU index for
  torch on Linux x86_64.
- Run `python scripts/ci/check_agents_md_size.py`. Upstream leaves little
  headroom under the root and chain caps, so fork lines in `AGENTS.md` must fit
  what remains.
- On a host with an NVIDIA GPU, run `hermes pm install` with the `discord`,
  `kittentts`, `stt-whisper`, and `onnx-cuda` extras. Generate speech through
  `text_to_speech_tool` with `tts.kittentts.backend: cuda` and confirm it runs
  on `CUDAExecutionProvider`, then transcribe it with `stt.local.device: cuda`.
- Restart the gateway on the merged code, confirm it connects to Discord, and
  run a real chat turn.

## Merge History

This is an append-only decision record, newest first, with one entry per upstream sync. It explains past integrations but does not by itself establish a current divergence; use the Divergence Log for that. Sync with upstream only when the user explicitly asks. Do not edit past entries.

### 2026-10-25 — Merge upstream/main into main

**Merge commit:** `ea0e444377611ac5f45064de5e6f300ff91b4f6a`, followed by the fork
fix-up commit that records this entry.
**Parents:** `3a917107d66913487f0182741e30fa8dbd3cf4ae` (fork) and
`4ea8d23cc9f527332ccb1d55359b27741b6ea51b` (upstream/main).

- Merge 24,710 upstream commits directly into `main` with the owner's explicit
  consent. Seven textual conflicts, all in fork-touched files.
- Upstream absorbed HD004 (YAML `discord.allow_bots` with env precedence,
  through the generic `extra_or_secret` reader) and HD005 (the install workflow
  no longer has a schedule), so both entries are removed and upstream's code is
  taken.
- Keep HD001's intent reader beside upstream's new `adapter_config_readers`
  module, and keep HD002's username labels in the reworked thread-starter path.
  Upstream's new voice-channel speaker labels used display names; they now use
  account usernames so HD002 covers voice turns too. Message edit/delete events
  feed lifecycle hooks only and keep upstream behavior.
- Upstream moved production to a PM-managed Python 3.14 environment. Its
  KittenTTS 0.8.1 wheel has no `backend` argument, so HD003's constructor change
  would have crashed TTS. HD003 now rebuilds the ONNX session on CUDA instead,
  and adds the Linux x86_64 `onnxruntime-gpu` packaging plus faster-whisper CUDA
  preload described in its entry. With the owner's agreement this accepts a
  per-sync `uv.lock` regeneration.
- Accept upstream additions including PM, the gateway-owned session runtime,
  side questions, and the version bump to 0.22.5.

**Validation:** Broad subsets (`tests/gateway`, `plugins`, `tools`,
`hermes_cli`, `agent`, `pm`) ran 54,473 passed, 80 failed, 9 errors, 722
skipped across 5,494 files. Every failing file was rerun on a pristine
`upstream/main` checkout with the same virtualenv. All fail identically except
`test_discord_turn_prompt_parity.py`, the HD002 voice gap fixed above, and
`test_session_mutation_compress_managed.py`, a 300s timeout that passed on rerun.
The fork-specific and HD003 test files then passed (Discord 178 tests across
voice/parity files, plus 150 across TTS, PM, transcription and metadata).
Live: `hermes pm install` built the environment with the discord, kittentts,
stt-whisper, mcp, and onnx-cuda extras. `hermes-gateway.service` restarted onto
it under Python 3.14 and connected to Discord as `Aila#9114`. A real
`hermes chat` turn returned a model reply, generated TTS on
`CUDAExecutionProvider`, and listed T3 MCP tools. Local STT transcribed that
audio on CUDA. A Discord-originated message was not sent.

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
