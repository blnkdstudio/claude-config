---
title: R&D Autonomous Pipeline
type: playbook
created: 2026-05-15
updated: 2026-05-15
---

# R&D Autonomous Pipeline

Autonomous product development pipeline applicable to any project. User says "run rnd" to start. Runs continuously until user says "stop" or "halt".

## Core Principles

### 1. Agents as Compaction
Every phase runs inside a sub-agent. Agents get a fresh context, do their work, write results to files, and return a short summary. The full working context is discarded — only the summary lands in the main conversation.

The **main conversation is just an orchestrator** that barely grows. All heavy lifting happens inside agents that naturally shed their context on return. File-based state carries continuity, not conversation memory.

**No manual `/compact` needed.** Auto-compact fires naturally if context ever fills.

### 2. Continuous Pipeline, Not Cycles
There are no "cycles." The R&D process is a **continuous pipeline** where each role runs independently, connected only through the filesystem:

```
backlog/       →     tasks/       →     in-progress/     →     completed/
  ↑ CTOs               ↑ Tech Lead        ↑ Workers
  (always feeding)     (always planning)  (always building)
```

Each agent reads the current filesystem state and acts. No waiting for other phases to finish. The filesystem IS the coordination layer.

### 3. Model Tiering
Use the cheapest model that can do the job:
- **Haiku** — CTO review (read-only analysis, pattern matching)
- **Sonnet** — Tech Lead (planning, prioritization, task writing)
- **Opus** — Workers (complex multi-file implementation)

### 4. Lean Agent Prompts
Don't embed context in prompts. Point agents to files:
- "Read `docs/rnd/state.md` for current project state"
- "Read `docs/rnd/backlog/` for open items"
- "Read `docs/rnd/completed/` to avoid redoing work"

The files ARE the context. Smaller prompt = fewer input tokens.

## Folder Structure
```
docs/rnd/
├── state.md          ← Current project state summary (~50 lines, updated by wrap-up)
├── backlog/          ← CTO drops findings (NNN-short-title.md)
├── tasks/            ← Tech Lead creates tasks (TNNN-short-title.md)
├── in-progress/      ← Workers move tasks here while building
├── completed/        ← Done tasks with implementation notes
└── log/              ← Pipeline logs (log-NNN.md)
```

## Pipeline Roles

### CTO Agents (background, continuous)
Two parallel agents that continuously feed the backlog:
1. **Internal Review** — scans codebase for bugs, UX issues, security, data integrity, performance
2. **Market Research** — researches competitors, integrations, industry trends

Rules:
- Read `state.md` + existing backlog + completed to avoid duplicates
- Write findings to `backlog/` with next available number
- **Don't wait** for Tech Lead or Workers — just keep feeding the backlog
- Skip this phase entirely if backlog already has 10+ unworked items

### Tech Lead Agent (triggered when backlog has items)
Converts backlog items into actionable tasks:
- Reads ALL `backlog/` items + `completed/` to avoid redoing work
- Reads relevant source code to understand current state
- Picks TOP 3 items (priority: bugs > security > UX > features)
- Creates detailed task files with exact file paths, code snippets, done criteria
- Notes which tasks can share a worker agent (same file dependencies)
- **Can start as soon as Internal Review finishes** — doesn't need to wait for Market Research

### Worker Agents (triggered when tasks/ has items, max 3 parallel)
Implement the tasks:
- Each worker reads their task → reads source → implements → builds → verifies
- Group tasks that share module wiring into ONE worker
- Parallel workers must NOT modify the same files
- Use `isolation: "worktree"` for git branch isolation when workers touch nearby files

### Wrap-up Agent (after workers complete)
Finalizes the batch:
1. Build all projects to verify
2. Commit with descriptive messages, push
3. Move completed tasks to `completed/`
4. Update `state.md` with current project state
5. Write pipeline log entry

## Orchestrator Behavior
The main conversation is a lightweight loop:

```
while user hasn't said "stop":
    if backlog has < 10 unworked items:
        launch CTO agents (background)

    if backlog has unworked items and no tasks pending:
        launch Tech Lead agent

    if tasks/ has items:
        launch Worker agents (up to 3 parallel)

    when workers finish:
        launch Wrap-up agent

    # Main context stays lean — just agent launches + summaries
    # User can chat, review, or do other work throughout
```

The user is NEVER blocked. Agents run in background. The conversation remains available for discussion, debugging, or planning.

## Sizing Guidelines
- 3 tasks per worker batch (reduces conflicts, ships faster)
- 2-3 worker agents max running simultaneously
- CTO writes 5 internal + 3 market = 8 findings per run
- Tech Lead picks only top 3 from full backlog
- Skip CTO phase when backlog is already full

## Lessons Learned
- Worker agents need explicit file paths — never let them guess
- Group tasks that share imports/wiring into one worker to avoid merge conflicts
- After workers finish, verify API paths match between frontend and backend
- All state persists in `docs/rnd/` files — safe across auto-compacts
- Nothing important should live only in conversation context
- **Agents ARE compaction** — each agent call naturally sheds context on return
- Main conversation should only contain orchestration + summaries
- **Pipeline > Cycles** — no reason to batch into discrete cycles when agents can run continuously
- **User isn't blocked** — background agents free the conversation for other work
