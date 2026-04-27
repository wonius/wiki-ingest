# Wiki-Ops Configuration

## Quick Setup

Set two environment variables before using this skill:

```bash
export WIKI_ROOT=/path/to/your/wiki      # your Obsidian vault root
export RAW_ROOT=/path/to/your/raw        # directory containing papers/articles/books
```

Or put them in your shell profile (`~/.bashrc`, `~/.zshrc`) for persistence.

---

## Environment Variables

| Variable | Default | Required | Description |
|---|---|---|---|
| `WIKI_ROOT` | `./wiki` | No | Root directory of your Obsidian wiki vault |
| `RAW_ROOT` | `./raw` | No | Root directory containing raw source files |
| `WIKI_SCHEMA` | `${WIKI_ROOT}/../schema/LLM_WIKI_SCHEMA.md` | No | Path to your wiki schema file |

### Setting Variables

**One-shot (this session only):**
```bash
export WIKI_ROOT=$HOME/my-wiki
export RAW_ROOT=$HOME/my-wiki/raw
```

**Persistent (add to shell profile):**
```bash
echo 'export WIKI_ROOT=$HOME/my-wiki' >> ~/.bashrc
echo 'export RAW_ROOT=$HOME/my-wiki/raw' >> ~/.bashrc
source ~/.bashrc
```

---

## Directory Layout

Your `WIKI_ROOT` should contain:

```
wiki/               # WIKI_ROOT
├── index.md       # page count statistics (updated by skill)
├── log.md         # ingest history
├── hot.md         # frequently accessed pages
├── lint-reports/  # Op3 lint output goes here
│
├── entities/      # entity pages (*.md)
│   └── *.md
├── concepts/      # concept pages (*.md)
│   └── *.md
├── summaries/     # source summary pages (*.md)
│   └── *.md
├── comparisons/   # comparison pages (*.md)
│   └── *.md
├── sources/       # source reference pages (*.md)
│   └── *.md
├── questions/     # Q&A pages saved by Op4 (*.md)
│   └── *.md
└── meta/          # metadata pages (*.md)
    └── *.md
```

Your `RAW_ROOT` should contain:

```
raw/                # RAW_ROOT
├── papers/         # PDF files — academic papers
│   └── *.pdf
├── articles/       # Markdown articles
│   └── *.md
└── books/          # PDF books or extracted text directories
    ├── *.pdf
    └── extracted_books/*/
```

---

## Schema

The skill uses a wiki schema to validate page structure. By default it looks for:
```
${WIKI_ROOT}/../schema/LLM_WIKI_SCHEMA.md
```

If your schema is elsewhere, set `WIKI_SCHEMA` explicitly:
```bash
export WIKI_SCHEMA=/path/to/schema/LLM_WIKI_SCHEMA.md
```

---

## Verification

After setup, verify the skill loads:
```
hermes skills list | grep wiki-ops
```

If you see `wiki-ops` in the list, you're ready to go.
