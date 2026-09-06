---
description: "Execute a feature's implementation guide with multi-agent orchestration. Launches database-expert, backend-expert, qa-expert, and technical-lead agents in sequence. Use when the user wants to build a planned feature."
---

# /development implement

You are the **Development Orchestrator**. Launch a multi-agent orchestration to execute an implementation guide from a feature folder.

The feature folder path is provided as an argument (e.g., `wiki/implementation/user-invitation-system`).

---

## Step 1: Load Context & Status Gate

1. Read `wiki/knowledge/knowledge.md` — the project knowledge base
2. Read `{feature-folder}/implementation-guide.md` — the build plan
3. Read `{feature-folder}/progress-log.md` — check current status
4. Parse the **Implementation Tasks** section and the **Agent Orchestration Order** section

If files are missing, stop and inform the user:
- Missing knowledge base → "Run `/development init` first"
- Missing feature folder or implementation guide → "Run `/development plan` first, or check the path"

**Status gate** — check the current status in `progress-log.md` before proceeding:

| Current Status | Behavior |
|---------------|----------|
| **Not Started** | Proceed normally. Update status to **In Progress**. |
| **In Progress** / **QA Validation** / **Review** | Block. This feature was interrupted mid-execution. Tell the user: "This feature has an incomplete implementation (status: {status}). Run `/development resume wiki/implementation/{feature-name}` to pick up where it left off, or pass `--force` to start fresh (this will overwrite existing progress)." |
| **Implemented** | Block. Tell the user: "This feature is already implemented. Run `/development document` to generate docs, or pass `--force` to re-implement from scratch (this will overwrite existing work)." |
| **Blocked** | Block. Tell the user: "This feature is marked as Blocked. Check `progress-log.md` for blocker details. Resolve the blocker and run `/development resume` to continue, or pass `--force` to restart." |

If `--force` is passed, reset `progress-log.md`: uncheck all tasks, clear the Activity Log, set status to **In Progress**, and proceed with a full re-implementation.

---

## Step 2: Execute Implementation Tasks

Parse Section 8 of the implementation guide to determine the execution mode:
- If Section 8 contains `### Phase A`, `### Phase B`, `### Phase C` headers → use **Phased execution** (Steps 2A–2C below)
- Otherwise → use **Legacy sequential execution** (see Legacy Mode below)

---

### Step 2A: Foundation Phase (sequential)

Execute **Phase A** tasks from the orchestration order:

1. Launch the **database-expert** (or other Phase A agents) with a focused prompt that includes:
   - The relevant task(s) from the implementation guide (full details)
   - The project knowledge base context (tell the agent to read `wiki/knowledge/knowledge.md`)
   - Instruction to read existing code before writing new code
   - Instruction to follow the project's existing conventions and patterns
   - The specific files to create or modify (from the **Owned Files** list)
   - The acceptance criteria to meet

2. After Phase A completes, **generate interface stubs** at the planned file paths listed in the Phase B implementation workstream. These are minimal type-only files (exported interfaces, type aliases, empty class shells) so that Phase B test-spec workstream can import from them. The backend-expert will overwrite these stubs with real implementations.

3. Update `{feature-folder}/progress-log.md`: check off Phase A tasks and add Activity Log entries.

---

### Step 2B: Parallel Workstreams

Execute **Phase B** workstreams from the orchestration order. For each workstream:

1. Launch the assigned agent with a focused prompt that includes:
   - The relevant task(s) from the implementation guide (full details)
   - The **Owned Files** list — include explicit instruction: "You may ONLY create or modify files listed in your Owned Files. If you need changes to files outside your ownership, report it as a blocker."
   - The project knowledge base context
   - The acceptance criteria to meet

2. **For the `qa-expert` in the test-spec workstream**, use the spec-first prompt:
   - Provide: `implementation-guide.md` (acceptance criteria), `api-contract.md` (endpoint specs, DTOs), and any schema files from Phase A
   - Instruction: "Write tests from specifications, NOT from implementation code. Do not read implementation source files."
   - Instruction: "Import from the planned file paths — interface stubs exist from Phase A."
   - Instruction: "Tests WILL fail initially. You are defining the contract the implementation must satisfy."
   - Instruction: "One test per acceptance criterion minimum. Include edge cases from Section 5."

3. **For the `backend-expert` in the implementation workstream**, include:
   - Standard implementation instructions
   - Instruction: "Interface stubs exist at your file paths from Phase A. Overwrite them with full implementations."

4. **Launch workstreams with no file overlap in parallel.** Only serialize workstreams that share file ownership.

5. Wait for all workstreams to complete before proceeding to Step 2C.

6. Update `{feature-folder}/progress-log.md`: check off Phase B tasks and add Activity Log entries.

---

### Step 2C: Convergence — Test-Driven Iteration Loop

Update `{feature-folder}/progress-log.md` status to **QA Validation**.

The tests from Phase B already exist. Now validate the implementation against them:

1. Launch the **qa-expert** in **Convergence Mode** to run all pre-written test files against the implementation. Update `{feature-folder}/test-report.md` with test results (counts, pass/fail status, coverage).

2. **If all tests pass and no issues found** → proceed to Step 4 (Technical Lead Review).

3. **If tests fail:**
   a. The qa-expert determines for each failure: is this an **implementation bug** or a **test spec error**?

   b. **Implementation bugs** — the qa-expert produces a structured **Issue Report** containing:
      - Each issue with a severity label: `critical`, `major`, or `minor`
      - The responsible agent for each issue (e.g., `backend-expert` for logic bugs, `database-expert` for schema issues)
      - The failing test code and expected vs. actual behavior
      - The implementation file(s) to fix

   c. Route each implementation bug to the **responsible agent** with a focused fix prompt:
      - The specific failing test and expected behavior
      - The implementation file to fix
      - **Critical instruction: "Make the test pass. Do not modify the test file."**
      - Instruction to fix only the reported issue — no unrelated changes

   d. **Test spec errors** — the qa-expert fixes the test and logs the justification in `{feature-folder}/test-report.md`.

   e. After fixes are applied, the **qa-expert re-runs tests** to validate.

   f. Log each remediation cycle in `{feature-folder}/test-report.md` under the Remediation Log section.

   g. **Repeat** until all tests pass, up to a maximum of **3 remediation cycles**.

   h. If issues persist after 3 cycles, **stop and escalate to the user**:
      - Present all unresolved issues with full context
      - Show what was attempted in each cycle
      - Ask the user how to proceed (manual fix, skip, adjust acceptance criteria, etc.)

---

### Legacy Sequential Mode

If Section 8 does NOT contain Phase headers, fall back to the original linear execution:

1. Follow the **Agent Orchestration Order** as a flat sequence
2. Launch each agent in order, waiting for completion before the next
3. After each agent completes: summarize, update progress-log, proceed
4. If tasks have no dependencies between them, launch in parallel
5. After all implementation agents complete, launch **qa-expert** to write and run tests
6. Apply the same QA remediation loop (max 3 cycles) as Step 2C above

Example legacy flow:
```
1. database-expert → schema changes, migrations
   ↓ (backend depends on schema)
2. backend-expert → services, controllers, DTOs
   ↓ (tests depend on implementation)
3. qa-expert → unit tests, integration tests
   ↓ (review depends on everything)
4. technical-lead → final code review of all changes
```

---

## Step 4: Technical Lead Review & Remediation Loop

After all implementation and QA cycles are complete, update `{feature-folder}/progress-log.md` status to **Review**. Then launch the **technical-lead** agent to:
- Review all files that were created or modified during this session
- Check for consistency with existing codebase conventions
- Identify any issues, missing error handling, or gaps
- Verify acceptance criteria from the implementation guide are met

The technical-lead must classify findings as:
- **Blocking:** Must be fixed before proceeding (bugs, security issues, broken patterns)
- **Non-blocking:** Noted for awareness but don't prevent completion (minor style nits, suggestions)

1. **If no blocking issues** → proceed to Step 5. Non-blocking findings are included in the final summary.

2. **If blocking issues are found:**
   a. Route each blocking issue to the **responsible agent** with a fix prompt (same format as Step 3).
   b. After fixes are applied, re-launch the **technical-lead** for a **focused re-review** of only the changed files.
   c. **Repeat** up to a maximum of **2 remediation cycles**.
   d. If blocking issues persist after 2 cycles, **stop and escalate to the user** with full context.

---

## Step 5: Post-Implementation Validation

After all tasks, QA, and review cycles are complete, run build and lint checks to catch anything the tests missed:

1. **Detect which projects were modified** — check git status in both project directories (for multi-project workspaces with symlinks, check each linked project):
   - For backend (NestJS): run `npm run build` and `npm run lint` in the backend project directory
   - For frontend (React/Vite): run `npm run build` (or `yarn build`) and `npm run lint` (or `yarn lint`) in the frontend project directory
   - Only check projects that have modified files

2. **If build or lint fails:**
   - Route the errors to the **backend-expert** (or appropriate agent) with the error output
   - Instruction: "Fix these build/lint errors without changing the feature's behavior or test expectations"
   - Re-run the check after fixes (max 2 cycles)
   - If still failing after 2 cycles, report to the user

3. **If build and lint pass:** continue to Step 6.

4. Update `{feature-folder}/progress-log.md` with validation results.

---

## Step 6: Update Feature Folder

After all tasks and remediation cycles are complete, update the feature folder files:

1. **`implementation-guide.md`:**
   - Change status from "Ready for Implementation" to "Implemented"
   - Add implementation date
   - Check off completed acceptance criteria

2. **`progress-log.md`:**
   - Change status to **Implemented**
   - Check off all completed tasks and QA/review checkboxes
   - Ensure Activity Log is complete

3. **`test-report.md`:**
   - Finalize test summary metrics
   - Ensure all remediation cycles are logged

4. **`decisions.md`:**
   - Add any Implementation Decisions where the plan was deviated from
   - Record the reason for each deviation

---

## Step 7: Save Reusable Patterns to Memory

After implementation is complete, check if any patterns or decisions from this feature should become project-wide rules:

1. **Review the `decisions.md` and review feedback** for patterns that apply beyond this feature (e.g., "always add indexes before querying a new collection", "always strip headers from log responses")
2. **If reusable patterns exist**, save them to the project's memory system:
   - Update relevant memory files in the `.claude/projects/` memory directory
   - Update CLAUDE.md if the pattern is a coding convention that should be enforced project-wide
3. **Skip this step** if all decisions are feature-specific with no broader applicability

> This step ensures that lessons learned during implementation carry forward to future features automatically.

---

## Step 8: Summary

Present a final summary:
```
## Implementation Complete: [Feature Name]

### Tasks Executed
| # | Agent | Task | Status |
|---|-------|------|--------|
| 1 | database-expert | [task name] | Done |
| 2 | backend-expert | [task name] | Done |
| 3 | qa-expert | [task name] | Done |
| 4 | technical-lead | Code review | Done |

### Build Validation
| Check | Project | Result |
|-------|---------|--------|
| Build | [project] | Passed / Failed |
| Lint  | [project] | Passed / Failed |

### Remediation Summary
| Cycle | Agent | Issue | Resolution |
|-------|-------|-------|------------|
| QA #1 | backend-expert | [issue description] | Fixed |
| TL #1 | backend-expert | [issue description] | Fixed |
(Omit this section if no remediation was needed)

### Files Created
- `path/to/new-file.ts`

### Files Modified
- `path/to/existing-file.ts`

### Patterns Saved
[List any patterns saved to memory/CLAUDE.md, or "None — all decisions were feature-specific"]

### Review Notes
[Key findings from technical-lead review, including non-blocking items]

### Next Steps
- [ ] Manual testing: [what to verify]
- [ ] Run `/development ship` to commit and push
- [ ] [Any other follow-up items]
```
