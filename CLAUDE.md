# my-insane-claude-workflow — Project Guidelines

## Overview
Focused extraction of the `claude-workflow` launcher + Anthropic-compatible gateway from the MIT-licensed ultrathink project. Launches Claude Code through a local gateway; main model stays on Anthropic, all other (subagent) Claude ids route to Codex gpt-5.5.

## Stack
- Pure ESM Node.js (>=20). No build step — runs straight from `js/`.
- Deps: `express` (gateway HTTP), `dotenv` (env loading).

## Architecture
- `js/cli/claude-workflow.js` — CLI: preflight (claude/codex/login), starts gateway, sets `ANTHROPIC_BASE_URL`, spawns `claude`.
- `js/gateway/` — `server.js` (HTTP, `/v1/messages`), `config.js` (env + `loadGatewayConfig`), `model-routing.js` (which id → which provider), `codex-provider.js` (drives `codex app-server`), `anthropic-format.js`, `trace.js`.
- `js/utils/` — `safe-path.js`, `env-loader.js` (loads `~/.ultrathink.env` then project `.env`; parent env wins).

## Conventions
- Config via env (`ULTRATHINK_GATEWAY_*`), overridable in `~/.ultrathink.env`.
- Default main model: `claude-opus-4-8` (Anthropic passthrough). Subagents: Codex gpt-5.5, xhigh effort.

## Current State
- Extracted 2026-06; runs via global `claude-workflow` (npm link) or `node js/cli/claude-workflow.js`.
- Routing verified: opus → Anthropic; sonnet/haiku/other → codex gpt-5.5/xhigh.

## Conventions for commits
- Commit as Kastarter <iiik7n@gmail.com> (per-repo config). No Claude co-author trailers.
