# Open Southeners — AI Skills

A collection of Claude Code skills for working with [Open Southeners](https://opensoutheners.com)' open-source Laravel packages.

The skills bias toward **retrieving from the official docs and the installed package source** instead of relying on Claude's pre-trained knowledge — so they stay correct as the packages evolve.

## What's included

| Skill | Use it when |
|---|---|
| `laravel-apiable` | Authoring or reviewing JSON:API endpoints built with [`open-southeners/laravel-apiable`](https://docs.opensoutheners.com/laravel-apiable/) |
| `laravel-data-mapper` | Authoring or reviewing DTOs built with [`open-southeners/laravel-dto`](https://docs.opensoutheners.com/laravel-data-mapper/) (the `laravel-data-mapper` repo) |
| `extended-laravel` | Discovering existing utilities in [`open-southeners/extended-laravel`](https://docs.opensoutheners.com/extended-laravel/) before writing new ones, or reviewing code that uses the package |

Skills are auto-loaded by Claude Code based on their description — you don't need to invoke them by name. Claude will pick the right one when it sees relevant code, dependencies, or task descriptions.

## Installation

### As a Claude Code plugin

Add this repo as a plugin in your Claude Code settings, or clone it into a plugins directory recognized by Claude Code. The plugin manifest is at [`.claude-plugin/plugin.json`](./.claude-plugin/plugin.json), and skills are auto-discovered from [`skills/`](./skills/).

### Via `skills.sh`

The skill files at `skills/<name>/SKILL.md` are also compatible with [`skills.sh`](https://skills.sh/). Point it at this repo and install whichever skills you want individually.

## Recommended: install the GitBook MCP for richer docs access

All three packages publish their docs on GitBook. The skills work without any extra setup (they fall back to `WebFetch` against the public docs URLs and to reading the installed package source under `vendor/`), but they get noticeably better when Claude can query the docs through the **official GitBook MCP for published docs**.

Setup instructions: <https://gitbook.com/docs/ai-and-search/mcp-servers-for-published-docs>

GitBook exposes a per-space MCP endpoint at `<docs-url>/~gitbook/mcp`. Add one MCP server per package:

- `https://docs.opensoutheners.com/laravel-apiable/~gitbook/mcp`
- `https://docs.opensoutheners.com/laravel-data-mapper/~gitbook/mcp`
- `https://docs.opensoutheners.com/extended-laravel/~gitbook/mcp`

When the MCP is configured, each skill prefers it over `WebFetch`.

## Source-of-truth precedence

Each skill follows the same precedence rule when answering questions:

1. **GitBook MCP** — if installed
2. **`WebFetch`** against the public GitBook URL
3. **Installed package source and tests** under `vendor/open-southeners/...`

When docs and code disagree, the **code and tests win**. This matters most for `laravel-data-mapper`, whose docs are still being completed.

## Contributing

These skills live in version control and evolve with the packages they describe. When a package gains a feature or changes an idiom, update the matching `SKILL.md`. Keep skills thin: prefer pointing Claude at the docs and source over baking API surface into the skill itself (it goes stale).

## License

MIT — see [`LICENSE`](./LICENSE).
