---
description: Rebuild the memory index from all memory files
agent: build
---
Read all files in `~/.opencode/memory/`:
1. Scan `decisions/`, `lessons/`, `preferences/`, `handoffs/`, `projects/`
2. Extract title, kind, date, and tags from each file's convention line (`> date | kind | tags`)
3. Group entries by tags into Topic clusters (top 10 tags by count)
4. List 10 most recent entries by date
5. List projects from `projects/` directory

Regenerate `~/.opencode/memory/_index.md` with this format:

```markdown
# Memory Index
_N entries across N categories | last updated YYYY-MM-DD_

## Topics
- **tag** (N) — _short summary of recent entries_
- ...

## Recent
- YYYY-MM-DD | [kind] title
- ...

## Projects
- **name** → `projects/name.md`
- ...
```

Keep the index under 500 tokens. If it would exceed 500 tokens, compact older entries to single-line references.
