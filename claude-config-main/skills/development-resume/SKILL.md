---
description: "Diagnose and resume a feature implementation that was interrupted. Use when an implementation was cut short by token limits, crashes, or network loss."
---

# /development resume

You are the **Development Orchestrator**. Diagnose and resume a feature implementation that was interrupted (token limit, machine shutdown, crash, network loss, etc.).

The feature folder path is provided as an argument (e.g., `wiki/implementation/user-invitation-system`).

---

## Step 1: Diagnose State

Read all feature folder files to understand where things stopped:

1. Read `{feature-folder}/implementation-guide.md` — the full plan
2. Read `{feature-folder}/progress-log.md` — which tasks are checked off, last Activity Log entry
3. Read `{feature-folder}/test-report.md` — any test results recorded
4. Read `{feature-folder}/decisions.md` — any implementation decisions already made

---

## Step 2: Analyze Completion

Cross-reference the progress log against the actual codebase to build an accurate picture:

1. **For each task in the checklist:**
   - If checked — verify the files listed in that task actually exist and contain the expected implementation (grep for key functions, classes, or patterns). Mark as **Verified** or **Inconsistent**.
   - If unchecked — check if the files were partially created anyway (the agent may have written files but the progress log wasn't updated before interruption). Mark as **Not Started**, **Partially Done**, or **Done but Unlogged**.

2. **Determine the interruption point:**
   - Last completed task (from Activity Log timestamp)
   - Current phase: Implementation / QA Validation / Review
   - Any in-flight work (files that exist but aren't in the progress log)

3. **If the implementation guide uses Phase A/B/C orchestration**, determine the interrupted phase:
   - If no Phase A tasks are complete → resume from Phase A
   - If Phase A is complete but Phase B workstreams are incomplete → resume incomplete workstreams only (verify completed workstreams' files exist)
   - If Phase B is complete but tests haven't been run → resume from Phase C (convergence)
   - If convergence was interrupted mid-cycle → check `test-report.md` for the last cycle number and resume the remediation loop
   - If convergence passed but review was interrupted → resume from review

4. **Check for corruption or conflicts:**
   - Files that were partially written (syntax errors, truncated content)
   - Test files that reference implementations that don't exist yet
   - Import statements pointing to missing modules

---

## Step 3: Present Diagnosis

Present a clear status report to the user:

```
## Resume Diagnosis: [Feature Name]

### Interruption Point
- **Last completed step:** [Task N: task name] by [agent]
- **Last activity:** [date and action from Activity Log]
- **Phase at interruption:** [Implementation / QA / Review]

### Task Status
| # | Task | Planned Agent | Status | Notes |
|---|------|--------------|--------|-------|
| 1 | [task name] | database-expert | Verified | Files exist and look correct |
| 2 | [task name] | backend-expert | Partially Done | `service.ts` exists, `controller.ts` missing |
| 3 | [task name] | qa-expert | Not Started | — |

### Issues Found
- [Any corrupted files, partial writes, or inconsistencies — or "None"]

### Recommended Action
[What the orchestrator recommends — see Step 4]
```

Ask the user: **"Should I proceed with the recommended action, or would you like to adjust?"**

---

## Step 4: Execute Resume

Based on the diagnosis, take the appropriate action:

**Scenario A: Clean interruption (task boundary)**
The last task completed fully, next task hasn't started.
- Skip all verified-complete tasks
- Resume from the first uncompleted task
- Follow the normal implement flow from that point (Steps 2–6 from the implement skill)

**Scenario B: Mid-task interruption (partial work)**
A task was partially completed (some files exist, others don't).
- Launch the responsible agent with a **completion prompt** that:
  - Lists what already exists (don't recreate)
  - Lists what's missing (create these)
  - Includes the original task details and acceptance criteria
  - Instructs the agent to read existing partial work before writing
- After completion, continue with remaining tasks

**Scenario C: Corrupted state (broken files)**
Files have syntax errors, truncated content, or inconsistencies.
- Present the corrupted files to the user
- Recommend either: (a) delete and redo the affected task, or (b) let the responsible agent fix the corruption
- After user confirms approach, execute it
- Continue with remaining tasks

**Scenario D: QA/Review interruption**
Implementation tasks were done but QA or review was interrupted.
- Skip directly to the appropriate phase (Convergence or Review from the implement skill)
- Use existing test results if available

**Scenario E: Parallel workstream interruption (Phase A/B/C mode only)**
Some Phase B workstreams completed, others didn't.
- Verify completed workstreams' files exist and are consistent
- Launch only the incomplete workstreams' agents with their original prompts and owned files
- Once all workstreams are done, proceed to Phase C (convergence)
- If Phase B test-spec workstream completed but implementation didn't (or vice versa), resume only the missing workstream

**In all scenarios:**
- Update `progress-log.md` with a resume Activity Log entry: `[date] | orchestrator | Resumed | Picked up from Task N after interruption`
- Follow the same remediation loops, file updates, and summary format as the implement command
