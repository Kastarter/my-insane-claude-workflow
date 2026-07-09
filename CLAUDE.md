# my-insane-claude-workflow — Project Guidelines

## Overview
Personal build of `claude-workflow`, derived from the MIT-licensed `ultrathink` project (© 2024 Lorhlona). Launches Claude Code through a local Anthropic-compatible gateway; main model stays on Anthropic, subagent/lower-tier Claude ids route to Codex gpt-5.5. Also supports a shared gateway daemon for plain `claude` sessions.

## Stack
- Pure ESM Node.js (>=20). No build step — runs straight from `js/`.
- Deps: `express` (gateway HTTP), `dotenv` (env), `undici` (upstream HTTP).

## Architecture
- `js/cli/claude-workflow.js` — launcher CLI (preflight, starts gateway, sets `ANTHROPIC_BASE_URL`, spawns `claude`; passes through `--resume`/`-r`/`--continue`/`-c`/`--fork-session`/`--from-pr`/`--session-id`).
- `js/cli/claude-workflow-daemon.js` — shared gateway daemon (bin `claude-workflow-gateway`); fixed port via `ULTRATHINK_GATEWAY_DAEMON_PORT` (default 4318).
- `js/anthropic-gateway-index.js` — standalone raw gateway entry.
- `js/gateway/` — `server.js`, `config.js`, `model-routing.js`, `codex-provider.js` (drives `codex app-server`), `anthropic-format.js`, `proxy.js`, `workflow-config.js` (workflow route defaults), `trace.js`.
- `js/utils/` — `safe-path.js`, `env-loader.js`.
- `scripts/` — daemon control (`claude-workflow-daemon.sh`), shell hook (`claude-workflow-gateway.bashrc`).
- `test/gateway.test.js` — `npm test`. `npm run check` = node --check on all sources.

## Config
- Env precedence: parent env → project `.env` → `~/.claude-workflow.env` → `~/.ultrathink.env` (compat).
- This machine's live config is in `~/.ultrathink.env`: `ULTRATHINK_GATEWAY_MAIN_MODEL_ID=claude-fable-5`, subagent + codex reasoning effort `xhigh`, `ULTRATHINK_GATEWAY_REFUSAL_FALLBACK=claude-opus-4-8`.
- `ULTRATHINK_GATEWAY_REFUSAL_FALLBACK`: when set, Fable/Mythos passthrough requests opt into Anthropic server-side refusal fallback (beta `server-side-fallback-2026-06-01`) so a safety-classifier decline is re-served by that model in the same call. Gateway retries once without it on a 400, so it can't break requests. Implemented in `js/gateway/server.js` (`refusalFallbackTarget`/`buildAnthropicBody`); config field `anthropic.refusalFallbackModel`.
- `[1m]` suffix on a model id is a client-visible alias only; Anthropic passthrough sends the plain id upstream.

## Current State
- Synced to upstream 2026-07 (added daemon, proxy, workflow-config, undici, resume/continue passthrough).
- Second upstream sync 2026-07 (3-way merge of `3672d87..dc74f05`): GLM 5.2 gateway routing (`glm` provider, `ULTRATHINK_GATEWAY_GLM_*` / `ZAI_*` env), Codex context-pressure fixes (tool-result byte caps `ULTRATHINK_GATEWAY_CODEX_TOOL_RESULT_MAX_BYTES`, auto-compact token limit + scope), Codex Read tool hardening, `test/glm/live-glm-gateway.js`. My refusal-fallback feature merged clean alongside it. Suite: 127 pass.
- Sync method for future upstream commits: fresh clone upstream, per-file `git merge-file` (base = last-synced upstream SHA, ours = repo, theirs = new upstream), then re-verify identity (package.json/README/LICENSE) and scrub `yshaaban`.
- Local `claude-workflow` command runs from this repo via `~/.local/bin/claude-workflow` wrapper (pins nvm node 22).

## Commit convention
- Commit as Kastarter <iiik7n@gmail.com> (per-repo config). No Claude co-author trailers.
