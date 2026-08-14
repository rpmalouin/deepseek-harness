# DeepSeek Harness — Fork

This is a personal fork of [`deepseek-ai/deepseek-harness`](https://github.com/deepseek-ai/deepseek-harness).
It tracks upstream `master` and adds a set of agent-workflow scaffolding that
makes **code-review-graph** the default way every coding agent explores and
reviews this codebase.

Everything here is an addition on top of stock `deepseek-harness` — no upstream
behavior is changed. The fork bundles three layers of integration that are
useful together when using this checkout as a **delegated / embedded coding
harness**:

1. **Graph-first code review & exploration** — a `code-review-graph` knowledge
   graph wired into every coding agent surface so structural understanding beats
   grep/read scanning (token-cheaper, faster, surfaces callers/dependents).
2. **OpenRouter LLM routing** — the headless one-shot profile can route its model
   requests through OpenRouter to `deepseek/deepseek-v4-flash-0731`.
3. **Hermes/agent delegation** — the checkout is designed to be driven as a local
   coding sub-agent from Hermes (or any host that shells out to the `dsh` CLI),
   not just run standalone.

Each is documented below.

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

### 5. OpenRouter LLM routing

`dsh`'s LLM capability is an adapter seam. The base mounts two adapters:
`llm-deepseek` (DeepSeek-native wire, `DEEPSEEK_API_KEY`) and `llm-pi-ai`
(generic multi-provider with **shipped OpenRouter support**). Stock headless
runs use `deepseek-official` → `DEEPSEEK_API_KEY`.

This fork documents and supports routing **through OpenRouter** instead, so the
same checkout can run its one-shot tasks on `deepseek/deepseek-v4-flash-0731`
(the same model Hermes itself runs on) with only an `OPENROUTER_API_KEY`:

```
OPENROUTER_API_KEY=<key> pnpm dsh --profile headless --patch openrouter.patch.yml "<task>"
```

The runtime overlay (reproduced inline below so the mechanism is part of the
repo's documentation) registers an `openrouter` provider profile on the already
mounted `llm-pi-ai` adapter and points `agent-default-model` at it:

```yaml
# Route the headless profile's model requests through OpenRouter.
# Adapter: llm-pi-ai (shipped openrouter catalog entry).
- id: llm-pi-ai
  config:
    providers:
      openrouter:
        apiKeyEnv: OPENROUTER_API_KEY
        baseURL: https://openrouter.ai/api/v1
        models:
          - id: !!js "process.env.DSH_OPENROUTER_MODEL ?? 'deepseek/deepseek-v4-flash-0731'"
            name: DeepSeek-V4-Flash-0731
        retryPolicy:
          mode: normal
          backoff:
            initialDelayMs: 500
            maxDelayMs: 10000
            jitterRatio: 0.1

# Point the headless one-shot default at the OpenRouter route.
- id: agent-default-model
  config:
    provider: openrouter
    model: !!js "process.env.DSH_OPENROUTER_MODEL ?? 'deepseek/deepseek-v4-flash-0731'"
```

The model is overridable via `DSH_OPENROUTER_MODEL`. Because the key comes from
the environment (`apiKeyEnv`), the overlay itself holds no secret — but the live
patch file is a local-only artifact and is **not** shipped in this fork (see
below). Recreate it from the snippet above with your own env var if you want the
OpenRouter route.

### 6. Hermes / agent delegation

The checkout is designed to be driven not just standalone but as a **local
coding sub-agent** from Hermes (sibling to the `codex` / `claude-code`
delegation targets). Hermes holds a `deepseek-harness` skill that shells out to
the `dsh` CLI for one-shot tasks:

```
terminal(command="pnpm dsh --profile headless '<task>'", workdir="<checkout>", timeout=600)
```

Key properties this fork relies on:

- `pnpm dsh` is the **source-launch** script (tsx, no rebuild needed between
  runs), so delegation works against `apps/cli/src` directly.
- `--profile headless "<task>"` builds a fresh persisted Agent, prints the final
  answer on stdout, and exits `0` on `completed`, `1` otherwise — an easy,
  parseable contract for a host agent.
- The invoking directory is the workspace root, and `AGENTS.md` (this repo's
  agent rules, including the graph-first section above) is loaded into the model
  context.
- Optional `--patch <overlay>` layers an extra cordis.yml patch (e.g. the
  OpenRouter route above) on top of the profile.

This integration is an **environment/consumer-side** capability (it lives in the
host agent's skill library, not in this repo's code) — but it is a first-class
purpose of the fork, which is why this document records it. Building the
checkout (`pnpm install && pnpm run build`) is a one-time prerequisite for both
standalone and delegated use.

## What is deliberately NOT included

The following local-only, machine-specific files are intentionally **excluded**
from this fork (they are gitignored and never committed):

- `Deepseek-OpenRouter-key` — a private API key file.
- `openrouter.patch.yml` — the live local OpenRouter routing overlay (its
  mechanism is documented inline above; recreate it with your own env var).
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
