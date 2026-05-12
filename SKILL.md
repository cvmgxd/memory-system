---
name: memory-system
description: |
  Lightweight markdown-based persistent memory for AI coding agents.
  Global + per-project scope with zero dependencies.
  Use when setting up cross-session memory for OpenCode, or when the user
  asks about remembering preferences, decisions, or lessons across sessions.
---

# Memory System — Zero-Dependency Persistent Memory

A lightweight markdown-based memory system that gives AI coding agents persistent memory across all projects and sessions. No databases, no servers, no API keys.

## Architecture

```
~/.opencode/memory/
  _index.md                 ← The map — all topics, projects at a glance (~200 tokens)
  decisions/                ← Architecture/technical decisions (global)
  lessons/                  ← Bugs, gotchas, patterns (global)
  preferences/              ← User preferences, workflow rules (global)
  handoffs/                 ← Session-end summaries
  projects/
    <project-name>/
      overview.md            ← Project context, stack, key decisions
      decisions/             ← Project-specific decisions
      lessons/               ← Project-specific lessons
```

## Setup (one-time)

1. Create directories:
```bash
mkdir -p ~/.opencode/memory/{decisions,lessons,preferences,handoffs,projects,templates}
```
2. Copy `templates/` files into `~/.opencode/memory/templates/`
3. Add to `~/.config/opencode/AGENTS.md`:

```markdown
## Memory Protocol

**Boot (EVERY session — MANDATORY):**
1. Read `~/.opencode/memory/_index.md` — scan topics, recent, and project list (the map of ALL memories)
2. If current workspace matches a project, read `~/.opencode/memory/projects/<name>/overview.md`
3. If task matches a topic tag, grep `~/.opencode/memory/` and read 1-3 matched files
4. Present brief confirmation: total entries, any relevant context found

**During session (auto-write — no command needed):**
After a decision, bug fix, or new preference: write to `~/.opencode/memory/<decisions|lessons|preferences>/<slug>.md` then run `/memory index`.

**Session end:** Run `/memory handoff`

**Entry format:** `> date | kind | tags` convention line. Kinds: decision, lesson, preference, handoff, note, bug.
```

4. Copy `commands/*.md` into `.opencode/commands/`
5. Create initial `~/.opencode/memory/_index.md` (see template below)

## Boot Protocol (what the agent does)

Every session, the agent:
1. Reads `_index.md` — gets the map of ALL memories
2. Reads the current project's `overview.md`
3. Greps for relevant global lessons/preferences
4. Presents a brief confirmation

This costs ~200 tokens at boot. Full entries loaded on demand.

## Commands

| Command | Purpose |
|---|---|
| `/memory save` | Scan session for durable decisions/lessons/preferences and save them |
| `/memory index` | Rebuild `_index.md` from all memory files |
| `/memory handoff` | Write end-of-session summary |

## Entry Format

All memory files use a simple markdown convention:

```markdown
# Kind: Title here

> YYYY-MM-DD | kind | tag1, tag2

## Context
...

## Content
...
```

Kinds: `decision`, `lesson`, `preference`, `handoff`, `note`, `bug`
Tags: lowercase, space-separated (project names, tech, topics)
