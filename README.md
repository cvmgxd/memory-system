# Memory System — Zero-Dependency Persistent Memory for AI Coding Agents

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: OpenCode](https://img.shields.io/badge/Platform-OpenCode-purple)](https://opencode.ai)
[![Dependencies: 0](https://img.shields.io/badge/Dependencies-0-green)]()

**A lightweight markdown-based memory system that gives AI coding agents persistent memory across all projects and sessions. No databases. No servers. No API keys. No npm packages. Just files.**

## The Problem

I work across multiple projects — game engines (Godot, Unreal), programming tools (Olamma, Sage Writer), and random utilities. Every time I started a new coding session, my AI agent forgot everything:

- The Unreal lesson I learned last week in a *different* project doesn't carry over
- My preference for "never deliver untested code" needs to be repeated to every session
- DeepSeek Flash keeps forgetting to compile — and I have to remind it every time
- I want the agent to know I prefer GDScript for Godot prototypes but C++ for Unreal gameplay

I first built **RLM** — a 600-line Node.js MCP server with TF-IDF search, NDJSON storage, and a JSON-RPC transport. It worked, but it was fragile: 30-second MCP startup timeouts on Windows, a complex bootstrap protocol, cache invalidation bugs, and 677 mostly-irrelevant entries cluttering the search index.

I looked at the alternatives:

- **Claude Code's auto-memory** — per-project only. My Unreal lessons don't follow me to Godot.
- **Mem0** — Docker, Postgres, OpenAI API keys. 55k stars but needs a server, costs money per operation, can't run offline.
- **ClawMem** — SQLite + llama.cpp + GPU. Incredibly powerful, but I don't want to run 3 models on my GPU for a memory system.
- **Cline Memory Bank** — 6 canonical files, great concept, but per-project only and came with a lot of ceremony around software specs I don't need.

None of them solved the actual thing I wanted: **a global memory that follows me across all projects, with enough context awareness to know which lessons apply where, zero infrastructure, and a boot time measured in milliseconds.**

## The Solution

Drop the search engine. Drop the MCP server. Drop the database. The AI agent is already the smartest thing in the room — let it decide what's relevant.

A global markdown memory bank with a lightweight index. At session start, the agent reads `_index.md` (~200 tokens — the map of everything). It knows which project it's in, filtres context accordingly, and loads detail on demand via grep and file reads. It uses the same tools it already has — read, grep, write, edit — with zero extra infrastructure.

```mermaid
graph TD
    A[Session Start] --> B[Read _index.md<br/>~200 tokens]
    B --> C{Workspace<br/>matches project?}
    C -->|Yes| D[Load project overview<br/>~projects/name/overview.md]
    C -->|No| E[Proceed without project context]
    D --> F{Task matches<br/>topics in index?}
    E --> F
    F -->|Yes| G[Grep memory/<br/>Read 1-3 matched files]
    F -->|No| H[No memory needed<br/>Proceed with task]
    G --> I[Agent has full context<br/>~500 tokens total]
    H --> I

    I --> J[During session]
    J --> K{Learn something<br/>durable?}
    K -->|Decision made| L[Write memory/decisions/slug.md]
    K -->|Bug fixed| M[Write memory/lessons/slug.md]
    K -->|Preference detected| N[Write memory/preferences/slug.md]
    L --> O[Rebuild _index.md]
    M --> O
    N --> O

    O --> P[Session End]
    P --> Q[Write memory/handoffs/handoff-date.md]
    Q --> O
```

## Architecture

```
~/.opencode/memory/
  _index.md                  ← The map — all topics, projects at a glance (~200 tokens)
  decisions/                 ← Architecture/technical decisions (global)
  lessons/                   ← Bugs found, patterns learned, gotchas (global)
  preferences/               ← Coding style, workflow rules, requirements (global)
  handoffs/                  ← Session-end summaries
  projects/
    olamma/
      overview.md             ← Project context, stack, key decisions
      decisions/              ← Olamma-specific decisions
      lessons/                ← Olamma-specific lessons
    your-project/
      overview.md
      decisions/
      lessons/
```

### Why This Architecture

| Concern | How We Solve It |
|---|---|
| **Boot speed** | Only `_index.md` is read at boot (~200 tokens). Detail files loaded on demand. |
| **Cross-project awareness** | Global index lists ALL projects. LLM sees the map before drilling into a specific project. |
| **Project isolation** | Each project has its own `decisions/` and `lessons/`. Olamma bugs don't pollute Godot context. |
| **Global preferences** | Coding rules and workflow preferences live at the top level. Applied everywhere. |
| **Search** | `grep -r "keyword" ~/.opencode/memory/` — instant, no search engine needed. |
| **Portability** | Plain markdown. Works with OpenCode, Claude Code, Cursor, any agent that reads files. |
| **Zero infrastructure** | No Node.js process, no MCP server, no API keys, no Docker, no database. |

## Comparison

| | Memory System | RLM | Mem0 | Claude Auto-Memory | ClawMem |
|---|---|---|---|---|---|
| **Dependencies** | 0 | Node.js + MCP | Docker + Postgres + API key | Claude-only | Bun + llama.cpp + GPU |
| **Boot time** | ~5ms | ~90ms (was 30s timeout) | 10-30s (Docker) | Instant | 2-10s (GPU) |
| **Global + per-project** | Yes | No | No | No | No |
| **Cross-agent** | Yes | No | Yes | No | Yes |
| **Offline** | Yes | Yes | No (needs LLM API) | Yes | Yes |
| **Storage** | Markdown files | NDJSON | Postgres + pgvector | Single MEMORY.md | SQLite + embeddings |
| **Search** | grep + LLM | TF-IDF | Semantic + BM25 + entity | Sequential read | Hybrid RAG |

## Setup (5 minutes)

```bash
# 1. Clone the skill
git clone https://github.com/cvmgxd/memory-system.git

# 2. Create memory directories
mkdir -p ~/.opencode/memory/{decisions,lessons,preferences,handoffs,projects,templates}

# 3. Copy templates
cp memory-system/templates/*.md ~/.opencode/memory/templates/

# 4. Copy commands (optional, for slash commands)
cp memory-system/commands/*.md .opencode/commands/

# 5. Add the Memory Protocol to ~/.config/opencode/AGENTS.md
# (see SKILL.md for the full protocol block)

# 6. Create initial _index.md
cp memory-system/templates/_index.md ~/.opencode/memory/_index.md

# 7. Create your first project
mkdir -p ~/.opencode/memory/projects/my-project
echo "# My Project" > ~/.opencode/memory/projects/my-project/overview.md
```

## Core Concepts

### The Index (`_index.md`)

The boot-time cheat sheet. Always under 500 tokens. The agent reads this first and knows everything that exists in memory. It shows:

- **Topics**: Tag clusters with entry counts and summaries
- **Recent**: Last 10 entries by date
- **Projects**: Every project with a link to its overview

### Auto-Write

No commands to remember. The agent detects when something durable happened and saves it:

- Made an architecture decision → `decisions/`
- Fixed a non-trivial bug → `lessons/`
- You expressed a preference → `preferences/`
- Session ending → `handoffs/`

### Slash Commands

| Command | Purpose |
|---|---|
| `/memory save` | Scan session for anything worth saving |
| `/memory index` | Rebuild `_index.md` from all memory files |
| `/memory handoff` | End-of-session summary |

### Entry Format

Every memory file follows a simple convention:

```markdown
# Kind: Title here

> YYYY-MM-DD | kind | tag1, tag2, tag3

## Context / Why
...

## Content / Decision / What happened
...

## Impact / Prevention / Applies to
...
```

**Kinds**: `decision` | `lesson` | `preference` | `handoff` | `note` | `bug`

**Tags**: lowercase, space-separated. Use project names, tech stack keywords, topic descriptors.

## Keywords

AI agent memory, persistent memory, coding agent, OpenCode memory, Claude Code memory, Cursor memory, agent context, long-term memory, cross-session memory, project memory, AI coding assistant, developer memory, knowledge retention, memory bank, markdown memory, zero-dependency memory, local-first memory, AGENTS.md, agent instructions, agent skills, coding workflow, software engineering, AI pair programming.

## License

MIT — free to use, modify, and distribute.

## Author

Built for OpenCode by [@cvmgxd](https://github.com/cvmgxd)
