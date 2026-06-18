# my-insane-claude-workflow

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Launch the normal **Claude Code** TUI through a private local gateway. Your main
session stays on Anthropic, while **workflow subagents are routed to Codex
(GPT-5.5)** through your local `codex login`.

This is a focused extraction of the `claude-workflow` launcher and its
Anthropic-compatible gateway — nothing else. It's derived from the MIT-licensed
[ultrathink](https://github.com/yshaaban/ultrathink) project.

## How it works

`claude-workflow` starts a local Anthropic Messages-compatible gateway, points
the child Claude Code process at it (`ANTHROPIC_BASE_URL`), and routes requests
by model id:

- The **main model** (`claude-opus-4-8` by default) stays on **Anthropic**.
- **Every other Claude model id** (i.e. all workflow subagents) is routed to
  **Codex `gpt-5.5`** via local `codex app-server` + `codex login`.

The harness, prompts, and tools are all Claude Code's — only the model answering
routed subagent turns is swapped. The Codex/GPT label is metadata only.

```
Claude Code TUI
   │  (ANTHROPIC_BASE_URL → local gateway)
   ▼
local gateway  ──  claude-opus-4-8*  ─────────▶  Anthropic (passthrough)
               └─  every other id   ─── Codex ─▶  gpt-5.5 (xhigh)
```

## Requirements

- Node.js 20 or newer.
- `claude` (Claude Code) on PATH with local auth.
- `codex` on PATH and logged in (`codex login`) for the routed models.

## Install

```bash
git clone https://github.com/Kastarter/my-insane-claude-workflow.git
cd my-insane-claude-workflow
npm install
npm link            # exposes the global `claude-workflow` command
```

## Configure

Defaults live in code; override them in `~/.ultrathink.env` (loaded
automatically) or a local `.env`. See [`.env.example`](.env.example). The
recommended setup:

```bash
# ~/.ultrathink.env
ULTRATHINK_GATEWAY_MAIN_MODEL_ID=claude-opus-4-8
ULTRATHINK_GATEWAY_SUBAGENT_REASONING_EFFORT=xhigh
ULTRATHINK_GATEWAY_CODEX_REASONING_EFFORT=xhigh
```

This keeps **Opus 4.8** as the only Anthropic-passthrough model and runs all
routed subagents at **gpt-5.5 / xhigh**.

## Use

```bash
cd /path/to/your/project
claude-workflow
```

That opens the normal Claude Code TUI. Routing to Codex only happens when Claude
actually delegates — ask it to **"use a workflow"** / fan out subagents (or use
the `ultracode` keyword). Plain chat runs entirely on the main Opus model.

One-shot check:

```bash
claude-workflow "Use a workflow to delegate a tiny subagent task, then summarize what happened."
```

Routed subagent rows show a label like `codex:gpt-5.5/xhigh via …`, confirming
the route.

### Notes

- Interactive and one-shot launches default to `--dangerously-skip-permissions`
  (auto mode). Use `--no-yolo` (or `CLAUDE_WORKFLOW_SKIP_PERMISSIONS=false`) to
  restore permission prompts.
- Each launch grabs its own OS-assigned localhost port, so multiple folders can
  run it at once. Set `ULTRATHINK_GATEWAY_PORT` only if you need a fixed port.
- Do not export `ANTHROPIC_BASE_URL` yourself — the launcher sets it for the
  child Claude process after the gateway picks a port.

## Layout

```
js/
  cli/claude-workflow.js     # the launcher
  gateway/                   # Anthropic-compatible gateway + Codex routing
    server.js  config.js  model-routing.js
    codex-provider.js  anthropic-format.js  trace.js
  utils/
    safe-path.js  env-loader.js
```

## License

MIT — see [LICENSE](LICENSE).
