---
description: "Verify codebase health (lint, build, test), then commit and push to the current branch. Use when the user wants to ship their changes."
---

# /development ship

You are the **Development Orchestrator**. Verify the codebase is healthy, then commit all changes and push to the current branch.

An optional commit message can be provided as an argument (e.g., `ship feat: add user invitations`). If no message is provided, you will generate one from the changes.

---

## Step 0: Detect Project Structure

Before running checks, detect the workspace structure:
1. Check if the working directory contains symlinks to other project directories (multi-project workspace)
2. If it does, identify which linked projects have uncommitted changes using `git status` in each project directory
3. Run pre-flight checks in **each project that has changes** — not just the workspace root
4. If only one project has changes, run checks only in that project
5. For each project, detect the package manager (`yarn.lock` → yarn, `package-lock.json` → npm) and use the correct commands

---

## Step 1: Parallel Pre-flight & Documentation Checks

Launch all checks concurrently using sub-agents. Pre-flight and documentation are independent — run them at the same time to minimize wall-clock time.

### Agent Dispatch

Launch **up to 4 agents in parallel** (in a single message with multiple Agent tool calls):

| Agent | Type | Task | Blocking? |
|-------|------|------|-----------|
| **qa-expert** | Pre-flight: Lint & Build | Run lint and build sequentially in each project with changes. Report pass/fail with error output. | Yes — blocks commit |
| **qa-expert** | Pre-flight: Tests | Run `npm test` (or `yarn test`) in each project with changes. If no test config, report "skipped". Report pass/fail with error output. | Yes — blocks commit |
| **technical-lead** | Docs: Knowledge Base & Feature Check | (1) Check if `wiki/knowledge/knowledge.md` exists and is stale (>10 commits since generation date). If stale, refresh it. (2) Scan `wiki/implementation/` for features with status **Implemented** that lack `feature-flow.md`. Report findings. | No — informational, except knowledge base update |
| **technical-lead** | Docs: CLAUDE.md & README Sync | Check if changes include new files in `agents/`, `skills/`, or `hooks/`. If so, verify `CLAUDE.md` and `README.md` reflect them (correct counts, entries). Auto-fix if out of sync. | No — auto-fixes silently |

### Agent Prompts

**Pre-flight: Lint & Build** (qa-expert):
- For each project detected in Step 0: run `{pm} run lint` then `{pm} run build` sequentially (build depends on lint passing)
- Stop at first failure and return the full error output
- If both pass, return "Lint: Passed, Build: Passed" per project

**Pre-flight: Tests** (qa-expert):
- For each project detected in Step 0: run `{pm} test`
- If the project has no test configuration (`test` script missing or returns "no test specified"), report "Tests: Skipped (no test config)"
- If tests fail, return the full failure output
- If tests pass, return "Tests: Passed" with count if available

**Docs: Knowledge Base & Feature Check** (technical-lead):
- Check if `wiki/knowledge/knowledge.md` exists
  - If it does not exist → report "Knowledge base: N/A"
  - If it exists → read the generation date, run `git log --oneline --since="[date]" | wc -l`
    - If >10 commits stale → refresh the knowledge base (read current codebase state, update `wiki/knowledge/knowledge.md`)
    - If ≤10 → report "Knowledge base: Fresh"
- Scan `wiki/implementation/` for feature folders (if the directory exists)
  - For each folder: read `progress-log.md` for status, check if `feature-flow.md` exists
  - Report any features with status **Implemented** that lack documentation
  - If no undocumented features → report "Feature docs: All documented"

**Docs: CLAUDE.md & README Sync** (technical-lead):
- Run `git diff --name-only --cached` and `git status --short` to identify changed/new files
- If new files exist in `agents/`, `skills/`, or `hooks/`:
  - Read `CLAUDE.md` and verify the structure tree has correct counts and entries
  - Read `README.md` and verify the skills/agents tables are complete
  - If out of sync → update both files to reflect the current directory contents
  - Report what was updated (or "CLAUDE.md & README: In sync")
- If no structural changes → report "CLAUDE.md & README: N/A"

### Collecting Results

Wait for all 4 agents to complete. Then evaluate:

1. **If any pre-flight agent reports failure** → show the errors and stop. Tell the user to fix the issues first or run `/development ship --force` to skip pre-flight checks.
2. **If all pre-flight agents pass** → continue to Step 2.
3. **Documentation results** are included in the summary regardless of pre-flight outcome.

> **Note:** With `--force`, skip launching the two pre-flight agents entirely. Still launch both documentation agents.

---

## Step 1 (fallback): Sequential Pre-flight

If the project is simple (single project, no symlinks) and you prefer not to use sub-agents for overhead reasons, you may run pre-flight checks directly:

1. **Lint:** Run `{pm} run lint`. Fail → stop.
2. **Build:** Run `{pm} run build`. Fail → stop.
3. **Tests:** Run `{pm} test`. Fail → stop. No test config → skip.

Documentation agents should still be launched in parallel with these sequential checks.

---

## Step 2: Stage & Commit

1. Run `git status` and `git diff` to understand what changed (including any documentation updates from the doc agents).
2. Run `git log --oneline -10` to understand the repo's commit message style.
3. Stage all relevant changed files (avoid staging secrets like `.env`, credentials, etc.).
4. If a commit message was provided as an argument, use it.
5. If no message was provided, generate a concise commit message from the staged changes following the repo's existing commit style.
6. Create the commit.

---

## Step 3: Push

1. Determine the current branch name with `git branch --show-current`.
2. If the branch is `main` or `master`, **warn the user** before pushing:
   - "You're about to push directly to `{branch}`. Are you sure? (Use `--force` to skip this warning.)"
   - Wait for confirmation unless `--force` was passed.
3. Push to the remote: `git push -u origin {branch}`.
4. If the push fails (e.g., no upstream, rejected), report the error clearly.

---

## Step 4: Summary

Present a final summary:

```
## Shipped

### Pre-flight (qa-expert agents)
| Check   | Project   | Result  |
|---------|-----------|---------|
| Lint    | [project] | Passed  |
| Build   | [project] | Passed  |
| Tests   | [project] | Passed / Skipped |

### Documentation (technical-lead agents)
| Check | Result |
|-------|--------|
| Knowledge base | Updated / Fresh / N/A |
| Undocumented features | None / [list] |
| CLAUDE.md & README sync | Synced / N/A |

**Branch:** `{branch}`
**Commit:** `{short-hash}` — {commit message}
**Pushed:** origin/{branch}
```

---

## --force Flag

If `--force` is present in the arguments:
- **Skip pre-flight agents** — do not launch the two qa-expert agents. Go straight to Stage & Commit.
- **Documentation agents still run** — launch both technical-lead agents even with `--force`. Docs should always be current.
- **Skip the main/master branch warning** — push without confirmation.
- Mention in the summary that pre-flight checks were skipped.
