---
description: "Update the existing knowledge base with recent codebase changes. Use when the knowledge base is stale and needs refreshing."
---

# /development update

You are the **Development Orchestrator**. The user wants to update the existing knowledge base.

Re-read the existing `wiki/knowledge/knowledge.md` and the codebase, then update only the sections that have changed. Preserve any manual annotations or notes that were added by the team. Update the generation date.

## Step 1: Load Current State

1. Read `wiki/knowledge/knowledge.md` — if it doesn't exist, tell the user to run `/development init` first
2. Extract the generation date from the header
3. Run `git log --oneline --since="[generation date]"` to identify what changed
4. Run `git diff --name-only HEAD~$(git log --oneline --since="[generation date]" | wc -l)..HEAD` to get the list of changed files

---

## Step 2: Parallel Section Updates

Launch **3 agents in parallel** (in a single message with multiple Agent tool calls), each responsible for updating their domain sections of the knowledge base. Pass each agent the current `wiki/knowledge/knowledge.md` content and the list of changed files.

| Agent | Sections to Update | What to Check |
|-------|-------------------|---------------|
| **backend-expert** | Architecture, Module Map, API Surface, External Integrations | New/removed modules, new/changed routes, new integrations, updated middleware/guards |
| **database-expert** | Database, Background Processing, DevOps | New/changed models or schema, new migrations, new queues or jobs, changed Docker/CI config, new env vars |
| **technical-lead** | Tech Stack, Testing, Code Conventions, Key Commands, Project Overview | Dependency version changes, new test utilities, changed linter/formatter config, new scripts/commands |

Each agent prompt should include:
- The current content of their assigned sections from `wiki/knowledge/knowledge.md`
- The list of changed files (so they know where to focus)
- Instruction: "Only update what has actually changed. Return your sections in the same markdown format."
- Instruction: "Preserve any manual annotations or notes (comments or sections not in the original template)."
- Instruction: "If nothing changed in your sections, return them unchanged and report 'No updates needed.'"

---

## Step 3: Merge & Write

1. Wait for all 3 agents to complete
2. Merge their updated sections back into `wiki/knowledge/knowledge.md`, preserving the file's structure and any manual annotations
3. Update the generation date in the header
4. Summarize what was updated by each agent
