## Why This Fork Exists

This fork tracks `deepseek-ai/deepseek-harness` upstream `master` and adds workflow scaffolding that makes the harness more effective as a delegated coding engine. Nothing in upstream behavior changes — everything here is additive: three integrations that work together when this checkout is used as an embedded coding harness.

1. **Graph-first code review and exploration** — a code-review-graph knowledge graph is wired into every coding agent surface, so agents explore and understand the codebase structurally (callers, dependents, cross-module relationships) instead of scanning files linearly. Review becomes cheaper, faster, and more accurate.

2. **LLM routing: native DeepSeek key first, OpenRouter only as a text fallback** — by default `dsh` runs on the DeepSeek-native wire (`DEEPSEEK_API_KEY`), the working path for agentic tool-calling delegation, where the agent actually executes bash, file writes, and test runs and completes end to end (with `DSH_PERMISSION_MODE=danger-full-access` where bash is needed, since the `workspace-write` preset has no bash sandbox here). The headless one-shot profile can also be overlaid onto OpenRouter, defaulting to `deepseek/deepseek-v4-flash-0731`, but that route is **text-only**: the proxy passes DeepSeek v4's DSML tool-call tags through as plain text, so a routed agent never executes its tools and the run hangs without completing. Use the native key for anything that must touch the filesystem and reserve OpenRouter for no-tool text one-shots.

3. **Hermes and agent delegation** — this checkout is designed to be driven as a local coding sub-agent from Hermes, or any host that shells out to the `dsh` CLI, so the harness becomes a deterministic execution engine inside a larger orchestration system rather than only a standalone CLI.

# DeepSeek Harness

English | [中文](README.zh.md)

DeepSeek Harness (`dsh`) is an open-source agent harness developed by [DeepSeek AI](https://deepseek.com).

It is built on an **everything-is-a-plugin** architecture and powered by [Cordis](https://github.com/cordiverse/cordis), whose design is described in [_A Programming Paradigm for Spatiotemporal Composability_](https://arxiv.org/abs/2608.25512).

Documentation: [https://deepseek-harness.github.io/deepseek-harness/](https://deepseek-harness.github.io/deepseek-harness/)

## Developer preview

DeepSeek Harness is in _developer preview_ and iterating rapidly. **THERE WILL BE COMPATIBILITY-BREAKING CHANGES.**

Review the [safety notice](SAFETY.md) before running the project.

## Run

### Run from `npm`

Install `Node.js`, then run:

```sh
npx @deepseek-ai/dsh web
```

The command starts the Web UI at `http://127.0.0.1:3080` by default and opens it in the default browser for a local launch. An SSH launch only prints the host URL because the SSH client or editor owns the local forwarded address. Pass `--no-open` to run the server without opening a browser. See [Web UI guide](docs/user/guide/index.md).

### Run from source

To run from a repository checkout:

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

`pnpm run build` prepares the repository artifacts. `pnpm dsh web` uses those built artifacts without rebuilding.

## Community and support

- Submit feedback or bug reports through [GitHub Discussions](https://github.com/deepseek-ai/deepseek-harness/discussions).
- Add the [`dsh-plugin`](https://github.com/topics/dsh-plugin) topic to your plugin repository for discoverability.
- Join <a href="https://discord.gg/Ycq5dCaS4">DeepSeek Harness Discord community</a>.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Development

Start with the [development guide](docs/development.md) and [architecture documentation](docs/architecture.md).

For agents, follow [AGENTS.md](AGENTS.md).

## Citation

```bibtex
@misc{deepseek-harness2026,
  title={DeepSeek Harness: Everything is a Plugin},
  author={DeepSeek-AI},
  year={2026},
  publisher={GitHub},
  howpublished={\url{https://github.com/deepseek-ai/deepseek-harness}},
}
```

## License

[MIT](LICENSE)

Third-party dependencies and their licenses are disclosed in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
