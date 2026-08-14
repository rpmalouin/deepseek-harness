# DeepSeek Harness — Fork

This is a personal fork of [`deepseek-ai/deepseek-harness`](https://github.com/deepseek-ai/deepseek-harness).
It tracks upstream `master` and adds a set of agent-workflow scaffolding that
makes **code-review-graph** the default way every coding agent explores and
reviews this codebase.

Everything here is an addition on top of stock `deepseek-harness` — no upstream
behavior is changed. The fork's value is the "graph-first" developer experience:
instead of grep/read-driven exploration, every supported coding agent is wired to
ask a pre-built knowledge graph of the repository for structural context first.
That graph is token-cheaper and faster than file scanning, and it surfaces
callers, dependents, and test coverage that flat search cannot.

## What this fork implements

### 1. `code-review-graph` knowledge-graph integration (the core addition)

The `code-review-graph` MCP server (a Python tool that builds a queryable graph
of a repo's code communities) is registered as an MCP server and driven through
hundreds of surfaces so that agent user-facing rules tell every agent to use the
graph before touching the filesystem.

- **`.mcp.json`** — standard MCP registration (`python3 -m code_review_graph serve`).
- **`opencode.jsonc`** — OpenCode MCP registration.
- **`.qoder/mcp.json`** — Qoder MCP registration.
- **`.cursorrules`, `.windsurfrules`, `GEMINI.md`, `QODER.md`, `CODEBUDDY.md`** —
  per-tool Agent rules that instruct the agent to use the graph tools first.
- **`.kiro/steering/code-review-graph.md`** — Kiro steering rule.
- **`.gemini/settings.json` + `.gemini/hooks/*.sh`** — Gemini CLI hooks that
  report graph status at session start and incrementally update the graph after
  writes.
- **`.qoder/settings.json`, `.codebuddy/settings.json`** — hook-based graph
  updates on PostToolUse and status on SessionStart (using `git rev-parse` to
  locate the repo, so they work in any checkout).
- **`.github/code-review-graph.instruction.md`** — GitHub Copilot/Ruleset-style
  instruction file (applies to `**`).

All server invocations auto-detect the repository from the current working
directory; no machine-specific absolute paths are baked in, so the config works
from any clone location.

### 2. Graph-based agent skills

Reusable `.agents/skills/*` skills (also mirrored under `.codebuddy/` and
`.gemini/`) that encode graph-powered workflows:

- `explore-codebase` — navigate and understand codebase structure via the graph.
- `debug-issue` — systematically debug issues using graph-powered navigation.
- `refactor-safely` — plan and execute refactoring using dependency analysis.
- `review-changes` — structured, risk-aware code review via `detect_changes_tool`.

### 3. Agent rules host

- **`AGENTS.md`** — appended a "code-review-graph MCP tools" section so the
  graph-first mandate is part of the repository's canonical agent rules.

### 4. Ignore the local graph database

- **`.gitignore`** — added `.code-review-graph/` so each clone builds its own
  graph and the build artifact is never committed.

## What is deliberately NOT included

The following local-only, machine-specific files are intentionally **excluded**
from this fork (they are gitignored and never committed):

- `Deepseek-OpenRouter-key` — a private API key file.
- `openrouter.patch.yml` — a private OpenRouter routing overlay for local runs.
- `MEMORY.md` — a private local agent-memory file with machine-specific paths.

If you clone this fork, build the graph and install the MCP server yourself:

```sh
# install the code-review-graph tool (from its own repo / package),
# then build the graph for this checkout:
code-review-graph build --repo .
```

## Keeping in sync with upstream

```sh
git fetch upstream
git checkout master
git merge upstream/master
git push origin master
```

Upstream tracking: this fork starts from upstream `master` at
`47f943859bef` (2026-08-13). Add any new upstream work by merging forward from
`deepseek-ai/deepseek-harness`.
