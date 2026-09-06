---
description: "Development workflow orchestrator — routes to build, init, plan, implement, resume, document, update, status, list, ship, research, debug subcommands. Use when the user invokes /claude-config:development with a subcommand argument."
---

You are the **Development Orchestrator** — responsible for initializing and maintaining the shared development knowledge base that all agents (database-expert, backend-expert, qa-expert, technical-lead, cto) rely on.

The user ran: `/claude-config:development $ARGUMENTS`

---

## Routing

Based on the argument below, invoke the matching skill using the **Skill tool**. Pass any remaining arguments along.

**Full argument string:** `$ARGUMENTS`

| If argument... | Invoke this skill |
|---------------|------------------|
| is `build` | `claude-config:development-build` |
| is `init` | `claude-config:development-init` |
| is `plan` | `claude-config:development-plan` |
| starts with `implement` | `claude-config:development-implement` with args: everything after `implement` |
| starts with `resume` | `claude-config:development-resume` with args: everything after `resume` |
| starts with `document` | `claude-config:development-document` with args: everything after `document` |
| is `update` | `claude-config:development-update` |
| is `status` | `claude-config:development-status` |
| is `list` | `claude-config:development-list` |
| starts with `ship` | `claude-config:development-ship` with args: everything after `ship` |
| is `research` | `claude-config:development-research` |
| is `debug` | `claude-config:development-debug` |

**After invoking the skill:**
- The invoked skill contains its own complete instructions — let it handle everything
- If the command expects a feature folder path, it will be in the remaining arguments (e.g., `implement wiki/implementation/foo` → path is `wiki/implementation/foo`)
- If `--force` appears anywhere in the argument, pass it along

---

## If no argument or unrecognized argument

Respond with:

```
Usage: /claude-config:development <command>

Commands:
  /claude-config:development build                      — Start a new project from scratch (interactive discovery → scaffold → GitHub)
  /claude-config:development init                       — Initialize the project knowledge base (full codebase analysis)
  /claude-config:development plan                       — Interactive planning session → generates feature folder
  /claude-config:development implement <feature-folder> — Execute a feature's implementation guide with multi-agent orchestration
  /claude-config:development resume <feature-folder>    — Diagnose & resume an interrupted implementation
  /claude-config:development document <feature-folder>  — Post-implementation documentation suite
  /claude-config:development list                       — List all features and their current status
  /claude-config:development update                     — Update the knowledge base with recent changes
  /claude-config:development status                     — Show knowledge base status and staleness check
  /claude-config:development ship [message]             — Lint + build + test, then commit & push to current branch
  /claude-config:development research                    — Investigate a topic and produce a research brief
  /claude-config:development debug                       — Systematically diagnose and fix a bug

Or invoke subcommands directly:
  /claude-config:development-build
  /claude-config:development-init
  /claude-config:development-plan
  /claude-config:development-implement <feature-folder>
  /claude-config:development-research
  /claude-config:development-debug
  ...etc

Flags:
  --force                    — Override status gates (use with implement to restart, document to skip checks)

Feature folder structure:
  wiki/implementation/{feature-name}/
  ├── implementation-guide.md    # Technical build plan              (plan)
  ├── api-contract.md            # Endpoint specs for frontend       (plan → document syncs)
  ├── ui-consumption-guide.md    # How UI integrates with feature    (plan → document syncs)
  ├── test-report.md             # QA results & remediation log      (implement)
  ├── progress-log.md            # Task checklist & status tracking  (implement)
  ├── decisions.md               # Key decisions & rationale         (plan → implement)
  └── feature-flow.md            # End-to-end process documentation  (document)

Research folder structure:
  wiki/research/{topic-name}/
  ├── research-brief.md            # Structured findings & recommendation  (research)
  └── decisions.md                 # Research-driven decisions              (research)

Incident folder structure:
  wiki/incidents/{incident-name}/
  └── incident-note.md             # Root cause, fix, prevention           (debug)

Workflow:
  New project:
  0. /claude-config:development build                                         → Start from scratch (discovery → scaffold → GitHub)

  Existing project:
  1. /claude-config:development init                                          → Understand the codebase (first time)
  2. /claude-config:development update                                        → Refresh knowledge base (before planning)
  3. /claude-config:development plan                                          → Design the feature (interactive)
     ↳ Auto: knowledge base freshness check
     ↳ Auto: tech-lead + CTO review before finalizing
  4. /claude-config:development implement wiki/implementation/{feature-name}   → Build it with agents
     ↳ Auto: post-implementation build + lint validation
     ↳ Auto: save reusable patterns to memory
     /claude-config:development resume wiki/implementation/{feature-name}      → (if interrupted)
  5. /claude-config:development document wiki/implementation/{feature-name}    → Document it (optional)
  6. /claude-config:development ship [optional commit message]                 → Verify, commit & push

  * /claude-config:development research                                        → Investigate before deciding
  * /claude-config:development debug                                           → Diagnose & fix a bug (multi-agent)
```
