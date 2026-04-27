# wiki-ingest

> Obsidian wiki five-in-one knowledge operations skill for Hermes Agent

**wiki-ingest** is a plug-and-play skill for the [Hermes Agent](https://github.com/siliconbrain-ai/hermes-agent), providing five core operations for Obsidian-based knowledge bases: ingest, query, lint, save, and autoresearch.

## Features

| Operation | Description |
|-----------|-------------|
| **Ingest** | Scan raw files, chunk them, and embed into wiki |
| **Query-Wiki** | Semantic search across your wiki |
| **Lint** | Auto-fix frontmatter, YAML links, aliases, tags |
| **Save** | Create/update wiki entries from structured input |
| **Autoresearch** | Periodic ingestion based on source config |

## Quick Start

### 1. Install

```bash
git clone https://github.com/wonius/wiki-ingest.git ~/.hermes/skills/wiki-ingest
```

### 2. Configure

```env
WIKI_ROOT=./wiki
RAW_ROOT=./raw
WIKI_SCHEMA=./schema.json
```

### 3. Load

```
/skill wiki-ingest
```

## License

MIT
