# helios

A Claude Code plugin marketplace. Each plugin in this repo bundles one or more skills (and, eventually, commands / hooks / agents) under a single namespace.

## Install

Add the marketplace once, then install plugins individually.

```
/plugin marketplace add mayankdharwa/helios
/plugin install io@helios
```

For a local clone:

```
/plugin marketplace add /path/to/helios
```

## Plugins

| Plugin | Skills | Slash commands |
|--------|--------|----------------|
| `io`   | `develop-feature`, `develop-mini-feature` | `/io:develop-feature`, `/io:develop-mini-feature` |
| `karpathy` | `guidelines` | `/karpathy:guidelines` |

### `io`

`develop-feature` — three-phase exploration → spec → build process for medium-to-large backend features. Invoke as `/io:develop-feature`, or with `args="review"` to act as the Review Agent during build.

`develop-mini-feature` — lightweight explore → approach → build cycle for ~day-scale features touching 1–2 areas. Invoke as `/io:develop-mini-feature`, or with `args="review"` for the Review Agent. Graduates one-way into `develop-feature` when scope grows.

### `karpathy`

`guidelines` — behavioral guidelines derived from Andrej Karpathy's observations on LLM coding pitfalls. Auto-triggers when writing, reviewing, or refactoring code: surface assumptions and inconsistencies, write minimum code, make surgical changes. Invoke explicitly as `/karpathy:guidelines`.

## Adding a plugin

Each plugin lives in its own top-level directory with this shape:

```
<plugin-name>/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── <skill-name>/
        └── SKILL.md
```

Then add an entry to `.claude-plugin/marketplace.json`:

```json
{
  "name": "<plugin-name>",
  "source": "./<plugin-name>",
  "description": "..."
}
```

Skills inside a plugin appear as `/<plugin-name>:<skill-name>`.
