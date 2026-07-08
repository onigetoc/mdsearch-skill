---
name: mdsearch
description: Full-text search engine for Markdown files using MiniSearch. Index + fuzzy search, context snippets, LLM formatting, line numbers. Use when searching .md files in a local folder — Obsidian vaults, knowledge bases, documentation. Supports boosting (title/headings/text), prefix search, JSON output, reindex, and custom cache directory.
---

# mdsearch — Full-text search CLI for Markdown files

Local full-text search CLI for Markdown knowledge bases and Obsidian vaults. With automatic indexing, fuzzy search, contextual snippets, line references, and LLM-ready output.

## Structure

- `src/search-md.mjs` — main CLI (bin `mdsearch`)
- `src/index-md.mjs` — index builder (bin `mdindex`)
- `src/lib.mjs` — shared: directory walk, frontmatter parsing, signatures, snippets
- Dependency: `minisearch` (npm)

## Version, Help & Listing

```bash
mdsearch --version
mdsearch --help
mdsearch --list                # Lists all indexed files (auto-builds index if missing)
mdsearch --list --reindex      # Forces re-indexing before listing
```

*Note: `mdsearch --list` automatically handles index creation if it doesn't exist. Use `--reindex` if you suspect the index is stale.*


## Installation & test

`npm link` is already set. Commands `mdsearch` and `mdindex` are globally available.

```bash
# local dev
mdsearch "my query" [folder/path]      # optional: relative folder or absolute path
mdsearch --reindex [folder/path]
mdindex [folder/path]

# from npm (when published)
npx mdsearch-cli "my query" [folder/path]
npx mdindex [folder/path]
```

## Essential commands

```bash
# ── SEARCH ──
mdsearch "minisearch"                     # current dir (.), fuzzy=0.2
mdsearch "minisearrch" --no-fuzzy          # exact only
mdsearch "minisearch" --fuzzy 0.4          # wider fuzzy
mdsearch "minisearch" notes-test           # specific folder

# ── BOOST ──
mdsearch "term"                           # boost: title=3, headings=2, text=1
mdsearch "term" --boost-title 5 --boost-headings 3 --boost-text 1
mdsearch "term" --boost-title 1 --boost-headings 1 --boost-text 1

# ── CONTEXT / LIMIT ──
mdsearch "term" --limit 5
mdsearch "term" --context 0               # no snippet
mdsearch "term" --context 4               # 4 lines before/after

# ── OUTPUT FORMAT ──
mdsearch "term"                           # human-readable
mdsearch "term" --json                    # JSON with normalized scores + line
mdsearch "term" --context 4 --llm-context # compact for LLM (+ Confidence)

# ── PREFIX ──
mdsearch "minis" --prefix

# ── CACHE / INDEX ──
mdindex ~/my-notes                        # pre-build index
mdsearch "minisearch" ~/my-notes           # uses cache
mdsearch "term" --reindex                  # force rebuild
mdsearch "term" ~/my-notes --cache-dir .mycache
mdindex ~/my-notes --cache-dir .mycache

# ── LLM-READY (typical usage) ──
mdsearch "PI agent shell injection" ~/my-notes --context 4 --limit 3 --llm-context
```

## Key conventions

- Dirs starting with `.` are ignored (`.git`, `.obsidian`, `.mdsearch`), plus `node_modules`
- Cache in `.mdsearch/` inside the target folder (index.json + meta.json)
- Cache invalidated automatically on file add/remove/modify
- Fuzzy search on by default (`--fuzzy 0.2`). `--no-fuzzy` for exact
- Fields are normalized (NFD → strip diacritics, lowercase)
- Snippets + line numbers read on demand (not cached)
- Line numbers always shown (`line 42: → content`)

## Output formats

- **Terminal**: `line 42: → match` with arrow, normalized score 0.00-1.00
- **JSON**: `{ "path": "...", "line": 42, "score": 0.92, "snippet": "..." }`
- **LLM context**: compact, `Confidence: 0.92` to help the LLM filter results

## Score system (important for the agent)

Score is **normalized between 0 and 1** (1.00 = best result of the session). Useful in `--json` or via `Confidence:` in `--llm-context`.

## Agent workflow: search → read → synthesize

The snippet is a **hint**, not the final source. After each relevant result, **read the full file at the indicated line** using your file reading tool.

### Loop

```
STEP 1: mdsearch broad terms --limit 5 --context 2
        → identify relevant files + lines

STEP 2: for each result with score > 0.3:
        read file.md offset:line limit:30
        → verify actual context (not just the snippet)

STEP 3: if more details needed, mdsearch with more specific terms
        → back to STEP 2

STEP 4: synthesize with citations (path + line)
```

### Rules

- **Never cite a snippet without reading the file** — the snippet is truncated to `context` lines.
- **Always use the line number** to cite: `"according to X.md:42, ..."`
- **Ignore results with score < 0.3** (likely noise).
- **Stop early** if read files confirm the answer (1-2 passes often enough).
- **Stop after 5 passes max** — don't loop forever. If nothing found, say so honestly.

### Concrete example

User: *"what is this project about architecture-wise?"*

```
PASS 1 → mdsearch "architecture" --limit 5 --context 2
         → results: archon-harness-builder.md (score 0.95, line 3)
                     context-engineering.md (score 0.40, line 15)

PASS 2 → read archon-harness-builder.md offset:3 limit:40
         → "the project is a YAML workflow system"
         → inferred: new terms "harness PIV workflow"
         
         mdsearch "harness workflow" --limit 3 --context 4
         → read best result at found lines

PASS 3 → synthesize: path + line for each source
```

## Query adaptation (important for the agent)

Content can be in French, English, or mixed. **Do NOT copy the user's query verbatim**, unless explicitly instructed.

Instead:

1. **Derive relevant search terms** from the user's intent — rephrase, extract keywords, add synonyms.
2. **Detect the content language** — read a few files or check filenames.
3. **Translate search terms** to match the content language.
4. **Search in multiple languages** if the vault is multilingual.
5. **Use fuzzy matching** (`--fuzzy 0.2` by default).
6. **Stay within `.md` files** — native format.

**Golden rule:** Translate intent, not words. Like Google.
