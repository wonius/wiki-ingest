---
name: wiki-ingest
version: "3.2.0"
description: "Obsidian wiki five-in-one knowledge operations: Op1 Ingest (batch papers/articles/books), Op2 Query→Wiki (Q&A traceability), Op3 Lint (full-library consistency), Op4 Save (persist Q&A), Op5 Autoresearch (auto-expand gaps). Trigger keywords: ingest/batch/process papers (Op1), search/wiki/look up (Op2), lint/scan/health check (Op3), save/bookmark (Op4), autor/expand/deep-dive (Op5)."
updated: 2026-04-27
---

# Wiki Knowledge Operations — Five-in-One Operations

Five operations for managing an Obsidian wiki knowledge base: Ingest (batch processing), Query→Wiki (Q&A traceability), Lint (consistency scanning), Save (persist Q&A), and Autoresearch (knowledge gap filling).

> **v3.2（2026-04-27）**：Refactored for open-source. All SiliconBrain-specific paths replaced with `${WIKI_ROOT}` / `${RAW_ROOT}` env vars.

---

## Five Operations Overview

| Operation | Trigger Keywords | Output |
|---|---|---|
| **Op1: Ingest** | papers/articles/batch/ingest/reprocess | wiki/summaries/ new summaries + Propagation |
| **Op2: Query→Wiki** | search/wiki/look up/cite wiki/wiki-based | Reply content + [[wiki/xxx]] citation chain |
| **Op3: Lint** | lint/scan/health check/consistency | Report written to wiki/lint-reports/ |
| **Op4: Save** | save/bookmark/remember this | Current Q&A persisted to wiki/questions/ |
| **Op5: Autoresearch** | auto-expand/deep-dive/autor | New entity/concept/summary pages filling gaps |

---

# Op1: Batch Ingest

> Core principle: **Truth over speed.** Evidence must be fully extracted before summary writing begins.

## Trigger

Activated when user says "process papers", "batch ingest", "ingest articles", "reprocess" or similar.

## Gate 1 — Understand-First Gate

> **Core principle**: Evidence extraction must be complete before template filling begins. Never create a template with `{{placeholder}}` markers and fill in later.

**Must extract all five before writing any summary:**
- [ ] **Mechanisms (SOPs)**: core workflows, algorithmic steps, operational procedures
- [ ] **Numbers/thresholds**: specific values, parameter ranges, experimental results, statistics
- [ ] **Failure modes**: error types, boundary condition failure points
- [ ] **Scope boundaries**: applicable and inapplicable scenarios (tradeoffs)
- [ ] **Guardrails**: limitations, circuit breakers, defensive design

**Prohibited:**
- ❌ Creating a summary template with `{{placeholder}}` markers and filling in later
- ❌ Claiming "understood the core mechanism" before reading the full document
- ❌ Writing subjective inferences as evidence (mark with `[推断]` prefix if uncertain)

## Gate 2 — Source Gate

- [ ] Title, author, and year embedded in filename match the actual document content
- [ ] Raw path is correct and file exists
- [ ] Flag high-risk files (suspicious naming / historical mismatches / extraction anomalies) for priority human review

## Gate 3 — Evidence Gate

- [ ] Key claims (table data, core arguments, performance metrics) must be locatable in raw/extracted text
- [ ] Quoted passages must be verified word-for-word against the source
- [ ] If extracted text quality is insufficient, explicitly mark as "low confidence" — do not fabricate
- [ ] All inferential conclusions must be prefixed with `[推断]`

**Contradiction Cross-Check (Gate 3.5):**
After completing the above, proactively check whether new content contradicts existing wiki pages:
- [ ] Compare the summary's core conclusions against related entity/concept/summary pages
- [ ] If a contradiction is found: add a `## Contradictions` section at the end of the summary, explaining the conflict and your judgment
- [ ] If the contradiction spans multiple existing pages, append contradiction records to those pages too
- [ ] **Never** hide or ignore contradictions — integrity over consistency

## Gate 4 — Consistency Gate

- [ ] frontmatter `sources` field paths must match the `## Sources` section paths
- [ ] File must not contain placeholder statements ("placeholder"/"TODO"/"[待补充]")
- [ ] Source topic, filename, and summary title must be mutually consistent
- [ ] links field format is correct (`[[Name]]` not `[[Name,]`)

## Gate 5 — Propagation Gate (⚠️ Must execute synchronously — do not defer)

Immediately after generating each summary:
1. [ ] From the summary's `## New/updated entities` table, create missing entity pages
2. [ ] From the summary's `## New/updated concepts` table, create missing concept pages
3. [ ] Update `wiki/index.md` statistics if changed
4. [ ] Append this entry to `wiki/log.md`
5. [ ] Update related entries in `wiki/hot.md` if it exists

## Gate 6 — Hygiene Gate

**Encoding pollution check:**
- [ ] Must not contain `�` (U+FFFD replacement character) or garbled characters
- Scan: `grep -rn $'\xEF\xBF\xBD' ${WIKI_ROOT}/summaries/`

**raw ↔ summaries mapping verification:**
- [ ] Paths in frontmatter `sources` must have corresponding files in the raw/ directory
- [ ] Paths in `## Sources` section must match frontmatter

**Placeholder residue scan:**
- [ ] Scan: `grep -rinE "placeholder|TODO|待补充|Summarize the core|\[\[TODO\]\]" ${WIKI_ROOT}/summaries/`

## Source Type Differences

### Papers
- Scan: `${RAW_ROOT}/papers/*.pdf`
- Extract: pymupdf4llm → pdf_text_extract_robust.py → PyMuPDF fallback
- Required coverage: Abstract / Introduction / Method / Experiment / Conclusion

### Articles
- Scan: `${RAW_ROOT}/articles/*.md`
- Extract: direct read, no PDF extraction tools needed
- Required coverage: argument logic, data sources, conclusion scope

### Books
- Scan: `${RAW_ROOT}/books/*.pdf` / `${RAW_ROOT}/books/extracted_books/*/`
- Required coverage: table of contents, core arguments, key quotations, scope
- Source format: list both PDF original path and extracted text path

## PDF Tool Fallback Chain

```
pymupdf4llm → opendataloader-pdf → pdfplumber → PyMuPDF(fitz) → record as "extraction failed"
```

## Quality Standards

- Understand-first > template-first
- Content accuracy > format compliance
- Evidence traceability > prose fluency
- Propagation synchronous > propagation deferred
- Hygiene Gate must not be skipped
- Small batches, fast cycles > large batch堆积
- End of each batch: output "verified + unverified" checklist

---

# Op2: Query→Wiki (Q&A Traceability)

> Trace user Q&A back to specific wiki pages. Answers must be traceable to wiki pages.

## Trigger

Activated when user says "search/wiki/look up/wiki-based/cite wiki/from wiki" or similar.

## Decision Flow

```
User question
  ↓
Step 1: Understand question, identify core concept/entity/relationship
  ↓
Step 2: Search wiki/ directory for relevant pages
  ↓
Found relevant pages?
  ├─ Yes → Step 3: Read relevant pages, extract answer
  └─ No → Step 4: Expand search (synonyms/approximate names)
       ├─ Found → Step 3
       └─ Still not found → Explicitly tell user "no wiki content found", suggest adding it
  ↓
Step 5: Attach [[wiki/xxx]] citation chain to answer, label confidence level
```

## Step 2 Search Strategy

```bash
# Precise search (filename and title)
grep -rl "target-keyword" ${WIKI_ROOT}/

# Concepts directory
ls ${WIKI_ROOT}/concepts/ | grep -i "keyword"

# Entities directory
ls ${WIKI_ROOT}/entities/ | grep -i "keyword"

# Summary full-text search
grep -rn "keyword" ${WIKI_ROOT}/summaries/

# wiki root index search
grep -n "keyword" ${WIKI_ROOT}/index.md
```

## Step 4 Expanded Search (when not found)

- Synonyms: Harness → Agent, Planning → 规划
- English→Chinese: instruction-tuning → 指令微调
- Abbreviation expansion: LLM → Large Language Model
- Check comparison pages for mentions of the topic

## Answer Format

```
Answer content (synthesized from multiple relevant pages)

Source Traceability:
- [[wiki/entity-xxx]] — provides entity info (Confidence: High/Medium/Low)
- [[wiki/concept-xxx]] — provides concept explanation (Confidence: High/Medium/Low)
- [[wiki/summary-xxx]] — provides summary details (Confidence: High/Medium/Low)

⚠️ If no wiki page support: [This part is wiki-external supplement, Confidence: Low]
```

## Confidence Levels

| Level | Condition |
|---|---|
| High | Multiple independent pages consistently pointing to the same conclusion |
| Medium | 1 clear supporting page exists, but boundary conditions not fully covered |
| Low | Inferential content, no direct wiki page support, or page content itself has low confidence |

---

# Op3: Lint (Full-Library Consistency Scan)

> Perform comprehensive consistency scanning of the wiki knowledge base. Detect schema violations, orphaned links, missing fields, and more.

## Trigger

Activated when user says "lint/scan/health check/consistency check" or automatically after Op1 batch processing.

## Scan Items (execute in order)

### 3.1 Frontmatter Required Fields

| page type | Required fields | Special fields |
|---|---|---|
| **All** | `title`, `type`, `created`, `updated` | — |
| **entity** | — | `description` (optional), `aliases` (optional) |
| **summary** | `sources`, `links`, `tags` | `raw` (recommended), `pages` (optional) |
| **comparison** | `comparisons`, `comparisons_type` | `comparisons` array (required) |
| **source** | `url` | `author` (recommended), `date_published` (recommended), `confidence` (optional) |
| **question** | — | `related` (recommended), `sources` (recommended) |
| **meta** | — | `status` (optional) |
| **concept** | — | (uses standard generic fields) |
| **overview** | — | (uses standard generic fields) |

**Scan script:**
```bash
cd ${WIKI_ROOT} && python3 << 'EOF'
import yaml, os, re

issues = []
required_by_type = {
    'entity': ['title', 'type', 'created', 'updated'],
    'summary': ['title', 'type', 'created', 'updated', 'sources', 'links', 'tags'],
    'comparison': ['title', 'type', 'created', 'updated', 'comparisons', 'comparisons_type'],
    'concept': ['title', 'type', 'created', 'updated'],
    'overview': ['title', 'type', 'created', 'updated'],
    'source': ['title', 'type', 'created', 'updated', 'url'],
    'question': ['title', 'type', 'created', 'updated'],
    'meta': ['title', 'type', 'created', 'updated'],
}

def check_file(path):
    with open(path) as f:
        content = f.read()
    if not content.startswith('---'):
        return
    parts = content[3:].split('---', 1)
    if len(parts) < 2:
        return
    fm_text = parts[1].split('---')[0]
    try:
        fm = yaml.safe_load(f'---\n{fm_text}---')
    except:
        fm = {}
    page_type = fm.get('type', '')
    required = required_by_type.get(page_type, ['title', 'type', 'created', 'updated'])
    for field in required:
        if field not in fm or fm[field] is None or fm[field] == '':
            issues.append(f"{path}: missing required field '{field}' (type={page_type})")

for root, dirs, files in os.walk('.'):
    for f in files:
        if f.endswith('.md'):
            check_file(os.path.join(root, f))

for issue in sorted(issues):
    print(issue)
print(f"\nTotal issues: {len(issues)}")
EOF
```

### 3.2 Orphaned Wiki-Link Check

Check whether `[[Target]]` links actually point to existing pages:

```bash
cd ${WIKI_ROOT} && python3 << 'EOF'
import re, os, glob

# Collect all pages that actually exist (without extension)
all_pages = set()
for f in glob.glob('entities/*.md'):
    all_pages.add(os.path.splitext(f)[0].replace('\\', '/'))
for f in glob.glob('concepts/*.md'):
    all_pages.add(os.path.splitext(f)[0].replace('\\', '/'))
for f in glob.glob('summaries/*.md'):
    all_pages.add(os.path.splitext(f)[0].replace('\\', '/'))
for f in glob.glob('comparisons/*.md'):
    all_pages.add(os.path.splitext(f)[0].replace('\\', '/'))
for f in glob.glob('sources/*.md'):
    all_pages.add(os.path.splitext(f)[0].replace('\\', '/'))
for f in glob.glob('questions/*.md'):
    all_pages.add(os.path.splitext(f)[0].replace('\\', '/'))
for f in glob.glob('meta/*.md'):
    all_pages.add(os.path.splitext(f)[0].replace('\\', '/'))

broken = []
for root, dirs, files in os.walk('.'):
    for f in files:
        if not f.endswith('.md'):
            continue
        path = os.path.join(root, f)
        with open(path) as fh:
            content = fh.read()
        links = re.findall(r'\[\[([^\]|]+)(?:\|[^\]]+)?\]\]', content)
        for link in links:
            link = link.split('#')[0]
            if link not in all_pages and link + '.md' not in all_pages:
                if f'/{link}' not in all_pages and f'{link.replace("-", "")}' not in all_pages:
                    broken.append(f"{path}: broken link [[{link}]]")

for b in sorted(broken):
    print(b)
print(f"\nTotal broken links: {len(broken)}")
EOF
```

### 3.3 Placeholder Residue Scan

```bash
grep -rinE "placeholder|TODO|待补充|Summarize the core|\[\[TODO\]\]" ${WIKI_ROOT}/
```

### 3.4 Encoding Pollution Scan

```bash
grep -rn $'\xEF\xBF\xBD' ${WIKI_ROOT}/
```

### 3.5 Frontmatter Sources Path Validation

```bash
cd ${WIKI_ROOT} && python3 << 'EOF'
import yaml, os, glob

issues = []
for f in glob.glob('summaries/*.md'):
    with open(f) as fh:
        content = fh.read()
    if not content.startswith('---'):
        continue
    fm_text = content[3:].split('---')[1]
    try:
        fm = yaml.safe_load(f'---\n{fm_text}---')
    except:
        continue
    sources = fm.get('sources', [])
    if not sources:
        continue
    for src in sources:
        path = src.strip('[]')
        if path.startswith('[[raw/'):
            path = path[1:-1]
        if not os.path.exists(path):
            issues.append(f"{f}: sources path not found: {path}")

for i in sorted(issues):
    print(i)
print(f"\nTotal source path issues: {len(issues)}")
EOF
```

### 3.6 index.md Statistics Validation

```bash
cd ${WIKI_ROOT} && python3 << 'EOF'
import os, glob, re

entities = len(glob.glob('entities/*.md'))
concepts = len(glob.glob('concepts/*.md'))
summaries = len(glob.glob('summaries/*.md'))
comparisons = len(glob.glob('comparisons/*.md'))
sources = len(glob.glob('sources/*.md'))
questions = len(glob.glob('questions/*.md'))
meta = len(glob.glob('meta/*.md'))
total = entities + concepts + summaries + comparisons + sources + questions + meta

print(f"entities: {entities}")
print(f"concepts: {concepts}")
print(f"summaries: {summaries}")
print(f"comparisons: {comparisons}")
print(f"sources: {sources}")
print(f"questions: {questions}")
print(f"meta: {meta}")
print(f"total: {total}")

try:
    with open('index.md') as f:
        content = f.read()
    stats = re.findall(r'\*\*总页面数 / Total Pages\*\*：(\d+)', content)
    if stats:
        print(f"index.md reports total: {stats[0]}")
        if int(stats[0]) != total:
            print(f"⚠️ MISMATCH: index.md says {stats[0]}, actual {total}")
except FileNotFoundError:
    print("index.md not found")
EOF
```

## Output Format

After lint completes, write the report to:

```
${WIKI_ROOT}/lint-reports/lint-YYYYMMDD-operation.md
```

Report format:

```markdown
---
title: Lint Report YYYYMMDD
type: lint-report
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: []
links: []
tags: [lint]
---

# Lint Report — YYYYMMDD

## Executive Summary

| Check | Result | Issue Count |
|---|---|---|
| Frontmatter required fields | ✅/❌ | N |
| Orphaned wiki-links | ✅/❌ | N |
| Placeholder residue | ✅/❌ | N |
| Encoding pollution | ✅/❌ | N |
| Sources path validation | ✅/❌ | N |
| Index statistics validation | ✅/❌ | N |

## Detailed Issue List

(Each issue with file path, description, severity level)

## Fix Recommendations

(For each issue, give a fix approach or direction)
```

---

# Op4: Save (Persist Current Q&A)

> Save the current conversation's Q&A as a structured page under `wiki/questions/`, for later traceability.

## Trigger

Activated when user says "save/bookmark/remember this/这个问题记一下" or similar.

## Workflow

1. [ ] Extract the core question and complete answer from the conversation
2. [ ] If the answer cites wiki pages, extract related entity/concept/summary names for `related` field
3. [ ] Choose/create page by question type:
   - Similar question page exists → append answer variant, do not overwrite
   - None exists → create new page
4. [ ] Generate frontmatter + answer content, write to `${WIKI_ROOT}/questions/question-SLUG.md`
5. [ ] Update `${WIKI_ROOT}/index.md` (questions count +1)

## Frontmatter Template

```yaml
---
title: "Question: [core question]"
type: question
created: YYYY-MM-DD
updated: YYYY-MM-DD
related: []
sources: []
tags: [q-saved]
---

# [core question]

[Full answer body. May contain [[wiki/xxx]] citations. The body IS the answer — do not use an answer frontmatter field.]
```

> **Note**: For question pages, the answer is body content, not the frontmatter `answer` field (per schema convention). The body starts with the core argument and can expand in sections.

---

# Op5: Autoresearch (Auto-Expand Knowledge Gaps)

> Analyze the current knowledge base, discover blanks, and proactively fill them — a "follow-the-vine" style knowledge expansion.

## Trigger

Activated when user says "auto-expand/deep-dive/autor/扩展一下" or when Op2/Op4 execution reveals a significant knowledge gap.

## Workflow

1. [ ] **Scan gaps**: Compare ingested summaries against related entities/concepts, find important directions without summary coverage
2. [ ] **Priority ranking**: Score by "impact × accessibility of existing material", select Top 3 gaps
3. [ ] **Confirm intent**: Show Top 3 gaps to user, ask "which one(s) to dig into?"
4. [ ] **After user confirmation**: Execute full Op1 Ingest workflow for selected gaps (find material → extract → write summary → Propagation)
5. [ ] **Reverse supplement**: If new entities/concepts were created, check for incomplete comparisons that could be made

## Gap Discovery Examples

```
Gap examples:
1. [Transformer] entity exists, but no LLM Transformer architecture evolution summary
2. [RLHF] summary exists, but no DPO vs RLHF vs GRPO comparison
3. [H100] entity exists, but no GPU cluster training efficiency summary
```

---

# Related Files

- Schema (authority): `${WIKI_SCHEMA}` (set via env var, defaults to `${WIKI_ROOT}/../schema/LLM_WIKI_SCHEMA.md`)
- Summary template: `${WIKI_ROOT}/meta/templates/wiki/summaries/summary-SOURCE_NAME.md` (optional)
- Lint reports: `${WIKI_ROOT}/lint-reports/`

---

# Pitfalls & Lessons Learned

## ⚠️ `[[WikiLink]]` YAML Syntax Pitfall

### Root Cause

Standard YAML (`yaml.safe_load`) parses `[[WikiLink]]` as nested arrays `[[['WikiLink']]]`, but Obsidian has a custom parser that correctly handles `[[WikiLink]]` as wiki-links.

Since Obsidian local viewing is the primary use case, the solution is: **lint checks use regex to extract links fields**, bypassing the YAML conflict.

### Correct Format (Obsidian Native)

```yaml
links:
  - [[上下文压缩]]
  - [[ACON]]
  - [[Agent Training]]
```

### Lint Implementation (fixed)

```python
import yaml, glob, re

errors = []
for f in glob.glob('entities/*.md') + glob.glob('concepts/*.md') + glob.glob('summaries/*.md'):
    with open(f) as fh:
        content = fh.read()
    if content.startswith('---'):
        # Use regex to extract links field (avoids [[]] wiki-link vs YAML conflict)
        links_pattern = re.findall(r'^\s+-\s+\[\[([^\]]+)\]\]', content, re.M)
        nested = [l for l in links_pattern if ',' in l or l.startswith('[')]
        if nested:
            errors.append(f"{f}: links nested — {nested}")
        # Check YAML overall parseability (excluding links field interference)
        parts = content.split('---', 2)
        try:
            fm = yaml.safe_load(parts[1])
        except Exception as e:
            errors.append(f"{f}: YAML parse error — {str(e)[:60]}")
```

> **Note**: `yaml.safe_load` does not error on unquoted `[[WikiLink]]` but returns nested arrays `[[['WikiLink']]]`. The lint's `isinstance(item, list)` check has been replaced by the regex approach. Other frontmatter fields (type, sources, title) still use standard `yaml.safe_load` for validation.

### Detection Script

```python
import yaml, glob, re

errors = []
for f in glob.glob('entities/*.md') + glob.glob('concepts/*.md') + glob.glob('summaries/*.md'):
    with open(f) as fh:
        content = fh.read()
    if content.startswith('---'):
        links_pattern = re.findall(r'^\s+-\s+\[\[([^\]]+)\]\]', content, re.M)
        nested = [l for l in links_pattern if ',' in l or l.startswith('[')]
        if nested:
            errors.append(f"{f}: links nested — {nested}")
        parts = content.split('---', 2)
        try:
            fm = yaml.safe_load(parts[1])
        except Exception as e:
            errors.append(f"{f}: YAML parse error — {str(e)[:60]}")

print(f"Problem files: {len(errors)}")
for e in errors[:5]:
    print(f"  {e}")
```

### Batch Fix Regex

```python
import re, glob

for f in glob.glob('entities/*.md') + glob.glob('concepts/*.md') + glob.glob('summaries/*.md'):
    with open(f) as fh:
        content = fh.read()

    # Fix comma-separated unquoted multi-items
    if re.search(r'^links:\s*\[\[.+?\]\]\s*,', content, re.MULTILINE):
        content = re.sub(
            r'^links:\s*\[\[([^\]]+)\]\](?:\s*,\s*\[\[([^\]]+)\]\])*',
            lambda m: '\n'.join(['links:'] + [
                f'  - "[[{i}]]"' for i in re.findall(r'\[\[([^\]]+)\]\]', m.group(0))
            ]),
            content,
            flags=re.MULTILINE
        )

    # Fix comma-separated single-item (no newline)
    content = re.sub(
        r'^links:\s*\[\[([^\]]+)\]\]\s*$',
        lambda m: f'links: [["[[{re.findall(r"\[\[([^\]]+)\]\]", m.group(0))[0]}]]"]]',
        content,
        flags=re.MULTILINE
    )

    with open(f, 'w') as fh:
        fh.write(content)
```

## ⚠️ Mixed WikiLink Path Styles

- `[[Page-Name]]` and `[[entities/Page-Name]]` should not be mixed in wiki internal references
- Prefer bare page names without directory prefixes (the system resolves to the correct directory automatically)
- Prefixes like `entities/` and `summaries/` are only for disambiguation scenarios

## ⚠️ `description` Is an Optional Field for Entities

- Schema defines `description` as an **optional** field for entities (does not block Lint)
- Do not block other legitimate operations because an entity lacks `description`
- But for new entities, adding a `description` when possible is recommended for readability
