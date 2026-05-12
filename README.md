# Memory System — Zero-Dependency Persistent Memory for AI Coding Agents

A lightweight markdown-based memory system that gives AI coding agents persistent memory across all projects and sessions.

**No databases. No servers. No API keys. Just markdown files.**

## Why

Every AI coding session starts blank. Your agent doesn't know what you decided last week, what bug you fixed yesterday, or that you prefer composition over inheritance. This fixes that — with nothing but markdown files and AGENTS.md rules.

## How it works

```
~/.opencode/memory/
  _index.md                 ← The map — all memories at a glance (~200 tokens)
  preferences/              ← Your preferences (global)
  lessons/                  ← Bugs and patterns learned (global)
  decisions/                ← Architecture decisions (global)
  handoffs/                 ← Session summaries
  projects/<name>/          ← Per-project context and decisions
```

At session start, the agent reads the index (200 tokens). It knows everything that exists. It loads details on demand.

## Setup

```bash
mkdir -p ~/.opencode/memory/{decisions,lessons,preferences,handoffs,projects,templates}
cp templates/*.md ~/.opencode/memory/templates/
```

Then add the Memory Protocol block to your `~/.config/opencode/AGENTS.md`.

## Commands

- `/memory save` — save durable context from the current session
- `/memory index` — rebuild the index from all entries
- `/memory handoff` — write end-of-session summary

## Built for OpenCode

Works with any AI coding agent that reads markdown. Designed for [OpenCode](https://opencode.ai).
