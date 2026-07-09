# mdsearch Skill

This skill provides an interface for interacting with the `mdsearch` CLI tool within the agent environment.

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
- `npm -y @onigetoc/mdsearch --version`: Check the installed version.
- `npm -y @onigetoc/mdsearch --help`: Display available CLI options.

### Listing Indexed Files
- `npm -y @onigetoc/mdsearch --list`: Lists all indexed files. (Auto-builds index if missing)
- `npm -y @onigetoc/mdsearch --list --reindex`: Forces a full re-index before listing files.

### Searching
- `npm -y @onigetoc/mdsearch "<query>"`: Search in the current directory.
- `npm -y @onigetoc/mdsearch "<query>" <folder>`: Search in a specific folder.
- `npm -y @onigetoc/mdsearch "<query>" --no-fuzzy`: Disable fuzzy matching.
- `npm -y @onigetoc/mdsearch "<query>" --phrase`: Exact phrase search (sequence of terms).
- `npm -y @onigetoc/mdsearch "<query>" --and`: Require all terms (default: OR).
- `npm -y @onigetoc/mdsearch "<query>" --prefix`: Enable prefix search.

### Formatting
- `npm -y @onigetoc/mdsearch "<query>" --json`: Output results in JSON format.
- `npm -y @onigetoc/mdsearch "<query>" --llm-context`: Output compact text optimized for LLM prompts.

## Agent Best Practices
- **Never cite a snippet directly**; always read the full file at the indicated line number to verify context.
- **Use `--llm-context`** when you need to pass search results into another prompt.
- **Stop early** once you have sufficient context to answer the user's question.
- **Translate user intent** into effective search terms rather than just copying their query.
