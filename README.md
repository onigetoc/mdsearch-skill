# mdsearch Skill

This skill provides an interface for interacting with the `mdsearch` CLI tool within the agent environment. It is based on [mdsearch](https://github.com/onigetoc/mdsearch), a fast full-text search CLI for Markdown knowledge bases.

Works with 🦞 [OpenClaw](https://www.clawhub.ai/therohitdas/youtube-full) (ClawdBot/Moltbot), <img src="assets/hermes.png" alt="" height="14"> [Hermes Agent](https://hermes-agent.nousresearch.com), Claude Code, Cursor, Antigravity, PI, Opencode and any agent that supports the [Agent Skills](https://skills.sh) format.

## Prerequisites

`mdsearch` must be installed. If not available, install it:

```bash
npm install -g @onigetoc/mdsearch    # global install (recommended)
npx -y @onigetoc/mdsearch "<query>"  # run without installing
```

If installed globally or locally in a project, all commands use `mdsearch` directly:

```bash
mdsearch "<query>"
```

If not installed, prefix with `npx -y @onigetoc/`:

```bash
npx -y @onigetoc/mdsearch "<query>"
```

## Overview

`mdsearch` is a fast full-text search CLI for Markdown knowledge bases and Obsidian vaults. It features:
- Automatic indexing
- Fuzzy search
- Contextual snippets with line references
- LLM-ready output formatting

## Workflow

The standard workflow for this skill is:
1. **Search**: Run `mdsearch "<query>" [folder]` to find relevant files.
2. **Read**: For highly relevant files (score > 0.3), read the content at the indicated line number.
3. **Synthesize**: Present the answer to the user citing sources (file path and line number).

## Commands

### Help & Version
- `mdsearch --version`: Check the installed version.
- `mdsearch --help`: Display available CLI options.
- `npx -y @onigetoc/mdsearch --version`: Via npx without install.

### Listing Indexed Files
- `mdsearch --list`: Lists all indexed files. (Auto-builds index if missing)
- `mdsearch --list --reindex`: Forces a full re-index before listing files.

### Searching
- `mdsearch "<query>"`: Search in the current directory.
- `mdsearch "<query>" <folder>`: Search in a specific folder.
- `mdsearch "<query>" --no-fuzzy`: Disable fuzzy matching.
- `mdsearch "<query>" --phrase`: Exact phrase search (sequence of terms).
- `mdsearch "<query>" --and`: Require all terms (default: OR).
- `mdsearch "<query>" --prefix`: Enable prefix search.

### Formatting
- `mdsearch "<query>" --json`: Output results in JSON format.
- `mdsearch "<query>" --llm-context`: Output compact text optimized for LLM prompts.

## Agent Best Practices
- **Never cite a snippet directly**; always read the full file at the indicated line number to verify context.
- **Use `--llm-context`** when you need to pass search results into another prompt.
- **Stop early** once you have sufficient context to answer the user's question.
- **Translate user intent** into effective search terms rather than just copying their query.

## Installation

### npx skills

```sh
npx skills add https://github.com/onigetoc/mdsearch-skill
```
