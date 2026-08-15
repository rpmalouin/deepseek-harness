## Why This Fork Exists

This fork tracks deepseek-ai/deepseek-harness upstream master and adds a set of workflow scaffolding that makes the harness far more effective as a delegated coding engine. Nothing in upstream behavior is changed — everything here is additive.
The fork layers three integrations that work together when using this checkout as an embedded coding harness:

1. Graph‑first code review & exploration
A code-review-graph knowledge graph is wired into every coding agent surface. Agents explore and understand the codebase structurally — callers, dependents, cross‑module relationships — instead of relying on grep or linear scanning. This makes review cheaper, faster, and dramatically more accurate.

2. OpenRouter LLM routing
The headless one‑shot profile can route its model requests through OpenRouter, using deepseek/deepseek-v4-flash-0731 by default. This enables consistent model behavior across environments and aligns the harness with modern OpenAI‑compatible gateways.

3. Hermes / agent delegation
This checkout is designed to be driven as a local coding sub‑agent from Hermes (or any host that shells out to the dsh CLI). The harness becomes a deterministic execution engine inside a larger orchestration system, not just a standalone CLI.

# DeepSeek Harness

English | [中文](README.zh.md)

DeepSeek Harness (`dsh`) is an open-source agent harness developed by [DeepSeek AI](https://deepseek.com).

It uses an architecture where **everything is a plugin**, and is powered by [Cordis](https://github.com/cordiverse/cordis), whose design is described in [_A Programming Paradigm for Spatiotemporal Composability_](https://github.com/cordiverse/paper).

## Developer preview

DeepSeek Harness is currently in _developer preview_ and is iterating rapidly. **THERE WILL BE COMPATIBILITY-BREAKING CHANGES.**

## Run

### Run from `npm`

Install `Node.js`, then run:

```sh
npx @deepseek-ai/dsh web
```

The command starts the Web UI, served at `http://127.0.0.1:3080` by default. See [Web UI guide](docs/user/guide/index.md).

### Run from source

To run from a repository checkout:

```sh
git clone https://github.com/deepseek-ai/deepseek-harness.git
cd deepseek-harness
pnpm install
pnpm run build
pnpm dsh web
```

## Community and support

- Feel free to submit feedback or bug reports through [GitHub Discussions](https://github.com/deepseek-ai/deepseek-harness/discussions).
- Add the [`dsh-plugin`](https://github.com/topics/dsh-plugin) topic to your plugin repository for discoverability.
- Join <a href="https://discord.gg/Ycq5dCaS4">DeepSeek Harness Discord community</a>.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Development

Start with the [development guide](docs/development.md) and [architecture documentation](docs/architecture.md).

For agents, follow [AGENTS.md](AGENTS.md).

## License

[MIT](LICENSE)

Third-party dependencies and their licenses are disclosed in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).
