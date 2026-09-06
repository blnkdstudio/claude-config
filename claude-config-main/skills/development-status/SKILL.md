---
description: "Check the knowledge base status and staleness. Use when the user wants to know if the knowledge base is up to date."
---

# /development status

You are the **Development Orchestrator**. The user wants to check the knowledge base status.

## Steps

1. Read `wiki/knowledge/knowledge.md` — if it doesn't exist, tell the user to run `/development init` first
2. Extract the generation date from the header
3. Run `git log --oneline --since="[generation date]"` to see how many commits have landed since
4. Scan for obvious staleness signals:
   - New modules in `src/modules/` not listed in the Module Map
   - New or removed dependencies in `package.json` not in Tech Stack
   - Changed routes not reflected in API Surface
5. Present a summary:

```
## Knowledge Base Status

- **Last generated:** [date]
- **Commits since:** [count]
- **Staleness:** [Fresh | Slightly Stale | Stale | Very Stale]

### Changes Detected
- [list of differences found, or "None — knowledge base appears up to date"]

### Recommendation
[Run `/development update` to refresh | Knowledge base is current]
```
