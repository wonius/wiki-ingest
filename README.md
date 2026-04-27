# wiki-ops

> Obsidian wiki five-in-one knowledge operations skill for Hermes Agent

**wiki-ops** is a plug-and-play skill for the [Hermes Agent](https://github.com/siliconbrain-ai/hermes-agent), providing five core operations for Obsidian-based knowledge bases: ingest, query, lint, save, and autoresearch.

## Features

| Operation | Description |
|-----------|-------------|
| **Ingest** | Scan raw files, chunk them, and embed into wiki |
| **Query→Wiki** | Semantic search across your wiki |
| **Lint** | Auto-fix frontmatter, YAML links, aliases, tags |
| **Save** | Create/update wiki entries from structured input |
| **Autoresearch** | Periodic ingestion based on source config |

## Quick Start

### 1. Install

```bash
# Clone into your Hermes skills directory
git clone https://github.com/wonius/wiki-ops.git ~/.hermes/skills/wiki-ops
```

### 2. Configure

Set these environment variables in your `~/.hermes/.env`:

```env
WIKI_ROOT=./wiki           # Obsidian wiki root
RAW_ROOT=./raw             # Raw source files root
WIKI_SCHEMA=./schema.json  # JSON Schema for frontmatter validation
```

See [CONFIG.md](CONFIG.md) for full configuration reference.

### 3. Load & Use

```
/skill wiki-ops
```

Then trigger operations by their keywords:

- **Ingest** — `ingest`, `ingest paper`, `in ingest mode`
- **Query** — `query`, `search wiki`, `semantic search`
- **Lint** — `lint`, `fix frontmatter`, `yaml lint`
- **Save** — `save`, `save to wiki`, `create wiki entry`
- **Autoresearch** — `autoresearch`, `research`, `periodic ingest`

## Requirements

- **Hermes Agent** (any recent version)
- **Python 3.10+** with standard library only
- **Jina AI API key** (for embeddings)
- **Obsidian** vault (optional, for schema validation)

## Project Structure

```
wiki-ops/
├── SKILL.md                        # Main skill file (load with /skill)
├── CONFIG.md                       # Environment variable reference
└── references/
    └── configuration.md            # Full setup guide
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `WIKI_ROOT` | `./wiki` | Obsidian vault root |
| `RAW_ROOT` | `./raw` | Raw source files root |
| `WIKI_SCHEMA` | *(none)* | Path to JSON Schema for frontmatter validation |
| `JINA_API_KEY` | *(from env)* | Jina AI API key for embeddings |

## License

MIT
