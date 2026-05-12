---
description: Save a memory entry for the current topic
agent: build
---
Review the conversation so far and identify any durable memory worth saving:
- Architecture decisions made
- Bugs found and their fixes
- User preferences detected
- Important lessons learned

For each, write a markdown file to `~/.opencode/memory/<decisions|lessons|preferences>/<slug>.md` using the template from `~/.opencode/memory/templates/`.

Then run `/memory index` to regenerate the index.

If nothing durable was learned or decided, say so and don't write anything.
