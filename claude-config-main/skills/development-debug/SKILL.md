---
description: "Systematically diagnose and fix a bug using multi-agent triage. The technical-lead triages, specialist agents investigate and fix, qa-expert verifies. Use when the user encounters a bug or unexpected behavior."
---

# /development debug

You are the **Development Orchestrator**. Run a systematic multi-agent debugging session to diagnose, fix, and verify a bug. The incident is documented in `wiki/incidents/`.

---

## Phase 0: Knowledge Base Freshness Check

Before starting the debugging conversation, silently check if `wiki/knowledge/knowledge.md` exists:
- If it **does not exist** → tell the user: "No knowledge base found. Run `/development init` first so debugging is grounded in the actual codebase."
- If it **exists** → read the generation date from the header. Run `git log --oneline --since="[date]" | wc -l` to count commits since. If more than 20 commits have landed, warn the user: "The knowledge base is stale ([N] commits since last update). Consider running `/development update` first for better context."

---

## Phase 1: Define the Symptom (interactive)

Begin by asking the user to describe the bug. Then ask targeted follow-up questions to fill in gaps. Do NOT proceed to Phase 2 until you have clarity on all of the following:

- **Symptom:** What exactly happens? What's the observed behavior vs. expected behavior?
- **Error output:** Any error messages, stack traces, HTTP status codes, log lines?
- **Reproduction:** Steps to reproduce, or conditions under which it occurs (always, intermittent, specific data, specific user?)
- **Scope:** When did it start? Was there a recent deploy, migration, config change? What was working before?
- **Impact:** How severe is this? (blocking users, data corruption, cosmetic, performance degradation)
- **Environment:** Production, staging, local? Which endpoint/page/flow?

Ask questions one round at a time — 2–3 focused questions per round. Do NOT dump all questions at once. Respond to the user's answers with your understanding before asking the next round.

---

## Phase 2: Evidence Gathering

Launch the **technical-lead** agent to perform the initial investigation. Include in the agent prompt:

1. The full symptom description and all context from Phase 1
2. Instruction: "Read `wiki/knowledge/knowledge.md` first for project context"
3. Instruction: "Identify the relevant module(s) and files from the symptom description"
4. Instruction: "Read the source code in the suspected area"
5. Instruction: "Run `git log --oneline -20 -- {relevant-files}` to check recent changes"
6. Instruction: "Run `git blame` on suspected lines if the symptom points to specific behavior"
7. Instruction: "Read related test files to understand expected behavior"
8. Instruction: "Check for related error handling patterns"
9. Instruction: "If logs or error output were provided, analyze them for patterns"

The technical-lead produces an **Evidence Report**:

```markdown
## Evidence Report

### Files Examined
| File | Relevance | Key Observations |
|------|-----------|-----------------|
| [path] | [why examined] | [what was found] |

### Recent Changes
| Commit | Date | Author | Files | Potential Relevance |
|--------|------|--------|-------|-------------------|
| [hash] | [date] | [author] | [files] | [assessment] |

### Code Analysis
[Detailed analysis of the suspected code path]

### Suspected Domain
[backend | database | API | auth | queue | frontend | infrastructure | configuration]

### Preliminary Hypothesis
[What the technical-lead thinks is happening based on evidence]
```

---

## Phase 3: Hypothesis Formation & Root Cause Analysis

Based on the Evidence Report, form 1–3 ranked hypotheses. For each hypothesis:

```markdown
### Hypothesis [N]: [Title]
- **Statement:** [Clear description of what might be wrong]
- **Evidence for:** [What supports this hypothesis]
- **Evidence against:** [What contradicts it, or "None"]
- **How to confirm:** [What would prove or disprove this]
- **Confidence:** [High / Medium / Low]
```

Present hypotheses to the user and ask: **"Does this match what you're seeing? Any additional context that might help narrow it down?"**

If the user provides new information, update hypotheses accordingly.

### Domain Routing Decision

Based on the confirmed hypothesis, determine which specialist agent should handle the fix:

| Suspected Domain | Primary Agent | When to Use |
|-----------------|---------------|-------------|
| Service logic, API behavior, integration | **backend-expert** | Business logic bugs, endpoint errors, external API issues |
| Database queries, schema, migrations, data | **database-expert** | Wrong query results, migration failures, data integrity |
| Test failures, test infrastructure | **qa-expert** | Flaky tests, test environment issues, coverage gaps |
| Architecture, cross-cutting concerns | **technical-lead** | Auth issues, middleware bugs, config problems, multi-module issues |

---

## Phase 4: Fix

### Single-Domain Bug

If the root cause is confined to one domain, route the fix to the appropriate specialist agent with a focused prompt including:

1. The confirmed hypothesis and root cause analysis
2. The specific file(s) to modify (from the Evidence Report)
3. The expected correct behavior (from Phase 1 symptom description)
4. Instruction: "Read `wiki/knowledge/knowledge.md` for project context"
5. Instruction: "Fix the root cause. Do not introduce workarounds."
6. Instruction: "Read the existing code before modifying. Understand the surrounding context."
7. Instruction: "Minimize the blast radius of your change. Fix only what is broken."
8. Instruction: "If you discover additional domains are involved, report it as a finding rather than modifying files outside your expertise."

### Multi-Domain Bug

If the root cause spans multiple domains (e.g., a schema issue AND a service logic issue), launch specialist agents **in parallel** — one per domain. Each agent gets:

- Its domain-specific portion of the root cause
- Its **owned files** (exclusive — no overlap between agents)
- The same base instructions as single-domain above

| Agent | Owned Files | Example |
|-------|-------------|---------|
| database-expert | `migrations/*`, `schema.*`, `prisma/*` | Schema fix, migration |
| backend-expert | `src/modules/**/*.service.ts`, `*.controller.ts`, `dto/*` | Logic fix, endpoint fix |
| qa-expert | `**/*.spec.ts`, `test/**` | Test infrastructure fix |

Wait for all parallel agents to complete before proceeding to Phase 5.

### Remediation Loop

**Max 2 cycles.** If the fix does not resolve the issue (determined in Phase 5), route back to the specialist with the failure details. After 2 failed cycles, escalate to the user:
- Present all attempted fixes with full context
- Show what was tried in each cycle
- Ask the user how to proceed (manual fix, workaround, adjust expectations, etc.)

---

## Phase 5: Verify Fix

Launch the **qa-expert** agent to verify the fix. Include in the agent prompt:

1. The original symptom description from Phase 1
2. The fix that was applied (files changed and what changed)
3. Instruction: "Run existing tests related to the affected module/files"
4. Instruction: "If a specific reproduction was described, verify the reproduction scenario is resolved"
5. Instruction: "Check for regression — ensure the fix doesn't break related functionality"
6. Instruction: "Run build and lint checks on affected files"

The qa-expert produces a **Verification Report**:

```markdown
## Verification Report

### Test Results
| Test Suite | Tests Run | Passed | Failed |
|-----------|-----------|--------|--------|
| [suite] | [count] | [count] | [count] |

### Reproduction Check
[Fixed / Not fixed / Partially fixed — details]

### Regression Check
[No regressions found / Regressions found: ...]

### Build & Lint
| Check | Result |
|-------|--------|
| Build | Passed / Failed |
| Lint  | Passed / Failed |
```

- **If verification passes** → proceed to Phase 6
- **If verification fails** → route back to Phase 4 (remediation loop, max 2 cycles)

---

## Phase 6: Write Incident Note

Derive the folder name from the incident (kebab-case, e.g., `payment-webhook-timeout`).

**Status gate:** Check if `wiki/incidents/{incident-name}/` already exists:
- If it exists and contains `incident-note.md` → warn the user: "An incident folder already exists at `wiki/incidents/{incident-name}/`. Continuing will overwrite existing files. Proceed? (or use a different incident name)"
- Wait for user confirmation before proceeding

Create the directory `wiki/incidents/{incident-name}/` if it doesn't exist. Generate the following file:

### File: `incident-note.md`

```markdown
# Incident Note: [Incident Name]

> Generated by `/development debug` on [date]
> Status: **Resolved**

---

## 1. Symptom

### Reported Behavior
[What the user observed]

### Expected Behavior
[What should have happened]

### Impact
- **Severity:** [Critical / High / Medium / Low]
- **Scope:** [How many users/flows affected]
- **Environment:** [Production / Staging / Local]

---

## 2. Timeline

| Time | Event |
|------|-------|
| [date] | Bug reported |
| [date] | Investigation started |
| [date] | Root cause identified |
| [date] | Fix applied |
| [date] | Fix verified |

---

## 3. Root Cause

### Summary
[One-paragraph root cause explanation]

### Technical Details
[Detailed technical explanation of what went wrong and why]

### Contributing Factors
- [Factor 1: e.g., missing validation, race condition, incorrect assumption]
- [Factor 2]

---

## 4. Fix Applied

### Changes Made
| File | Change | Agent |
|------|--------|-------|
| [path] | [what was changed] | [agent that made the fix] |

### Commits
| Hash | Message |
|------|---------|
| [hash] | [message] |

---

## 5. Verification

### Tests Run
| Test | Result |
|------|--------|
| [test name/file] | Passed / Failed |

### Reproduction Check
[Confirmed fixed / Still reproducing]

### Regression Check
[No regressions found / Regressions found: ...]

---

## 6. Prevention

### What would have caught this earlier?
- [ ] [e.g., Input validation on field X]
- [ ] [e.g., Integration test for scenario Y]
- [ ] [e.g., Monitoring alert on metric Z]

### Recommended Follow-ups
- [ ] [Action item 1]
- [ ] [Action item 2]

---

## 7. Lessons Learned

[Any broader insights — patterns to adopt, patterns to avoid, process improvements]
```

---

## Phase 7: Summary

Present a final summary:

```
## Debug Complete: [Incident Name]

### Symptom
[One-line description]

### Root Cause
[One-line root cause]

### Fix
| File | Change | Agent |
|------|--------|-------|
| [path] | [what was changed] | [agent] |

### Verification
| Check | Result |
|-------|--------|
| Related tests | Passed |
| Reproduction | Fixed |
| Regression | None |
| Build/Lint | Passed |

### Remediation Cycles
[N cycles needed — or "Fixed on first attempt"]

### Artifact
| File | Path |
|------|------|
| Incident Note | `wiki/incidents/{incident-name}/incident-note.md` |

### Prevention Recommendations
- [ ] [recommendation 1]
- [ ] [recommendation 2]

### Next Steps
- [ ] Run `/development ship` to commit the fix
- [ ] [Any other follow-ups]
```
