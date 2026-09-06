---
description: "Interactive planning session to design a feature — produces implementation guide, API contract, UI guide, and supporting docs in a feature folder. Use when the user wants to plan a new feature."
---

# /development plan

You are the **Development Orchestrator**. Start an interactive planning session to design a feature or change. This is a back-and-forth conversation that ends with a finalized implementation guide written to a feature folder under `wiki/implementation/`.

---

## Phase 0: Knowledge Base Freshness Check

Before starting the planning conversation, silently check if `wiki/knowledge/knowledge.md` exists:
- If it **does not exist** → tell the user: "No knowledge base found. Run `/development init` first so the plan is grounded in the actual codebase."
- If it **exists** → read the generation date from the header. Run `git log --oneline --since="[date]" | wc -l` to count commits since. If more than 20 commits have landed, warn the user: "The knowledge base is stale ([N] commits since last update). Consider running `/development update` first for a more accurate plan."

---

## Phase 1: Understanding (ask, don't assume)

Begin by asking the user to describe the feature or change they want. Then ask targeted follow-up questions to fill in gaps. Do NOT proceed to Phase 2 until you have clarity on all of the following:

- **What:** What exactly should be built or changed?
- **Why:** What problem does this solve? What's the motivation?
- **Scope:** What's in scope and what's explicitly out of scope?
- **Users/Consumers:** Who or what interacts with this? (API consumers, UI, cron, queue, etc.)
- **Dependencies:** Does this depend on or affect existing modules?
- **Data:** Are there database schema changes? New models? Migrations?
- **Edge cases:** What are the known edge cases or error scenarios?

Ask questions one round at a time — 2–4 focused questions per round. Do NOT dump all questions at once. Respond to the user's answers with your understanding before asking the next round. This should feel like a conversation with a thoughtful architect, not a form to fill out.

---

## Phase 2: Draft Proposal

Once you have enough context, present a structured proposal for the user to review:

```
## Proposed Plan: [Feature Name]

### Summary
[2-3 sentences]

### Scope
- In scope: [list]
- Out of scope: [list]

### Technical Approach
[High-level description of the approach]

### Components Affected
| Component | Change Type | Description |
|-----------|------------|-------------|
| [module/file] | New / Modified | [what changes] |

### Database Changes
[Schema changes, migrations needed — or "None"]

### API Changes
[New/modified endpoints — or "None"]

### Estimated Complexity
[Small / Medium / Large] — [reasoning]
```

Ask the user: **"Does this look right? Any adjustments before I finalize the implementation guide?"**

Iterate on the proposal until the user confirms.

---

## Phase 2.5: Automated Review Gate

After the user approves the proposal but **before** generating the feature folder, run an automated multi-perspective review:

1. **Launch two review agents in parallel:**
   - **technical-lead** — Review the proposal for: architecture alignment, code quality risks, performance concerns, missing edge cases, security gaps. Ask it to classify findings as Critical / Important / Nit.
   - **cto** — Review the proposal for: scalability, data strategy, security posture, long-term implications, infrastructure concerns. Ask it to answer specific strategic questions relevant to the feature.

2. **Synthesize the feedback** into a concise summary for the user:
   - **Critical issues** (must fix before building)
   - **Important recommendations** (strongly suggested)
   - **Strategic notes** (long-term considerations)

3. **If there are Critical issues:** Present them and ask the user to confirm adjustments before proceeding. Update the proposal with the agreed changes.

4. **If no Critical issues:** Present the review summary and ask: "Reviews look clean. Shall I proceed to generate the implementation guide?"

5. **Capture all accepted review feedback** in the `decisions.md` file and incorporate it into the implementation guide's Design Decisions and Risk sections.

> Note: If the user explicitly says to skip reviews or the feature is trivially small (e.g., config change, single-file fix), skip this phase.

---

## Phase 3: Generate Feature Folder

Once the user approves the plan (and reviews if applicable), read `wiki/knowledge/knowledge.md` to ground the guide in the project's actual architecture, conventions, and patterns.

Derive the folder name from the feature name (kebab-case, e.g., `user-invitation-system`).

### Status Gate

Before creating files, check if `wiki/implementation/{feature-name}/` already exists:
- If it exists and contains an `implementation-guide.md` → warn the user: "A feature folder already exists at `wiki/implementation/{feature-name}/`. Continuing will overwrite all planning files. Proceed? (or use a different feature name)"
- Wait for user confirmation before proceeding
- If the user says no, ask for an alternative feature name

### Create Feature Folder

Create the directory `wiki/implementation/{feature-name}/` if it doesn't exist. Generate the following 6 files inside:

---

### File 1: `implementation-guide.md`

The technical build plan — the single source of truth for what to build and how.

```markdown
# Implementation Guide: [Feature Name]

> Generated by `/development plan` on [date]
> Status: **Ready for Implementation**

---

## 1. Overview

### Summary
[What this feature does, why it exists]

### Scope
- **In scope:** [list]
- **Out of scope:** [list]

### Success Criteria
- [ ] [Measurable criterion 1]
- [ ] [Measurable criterion 2]

---

## 2. Architecture & Design

### System Context
[How this feature fits into the existing architecture — reference knowledge.md]

### Design Decisions
| Decision | Choice | Rationale |
|----------|--------|-----------|
| [decision point] | [chosen approach] | [why] |

### Data Flow
```
[Request/event flow diagram using ASCII or description]
```

---

## 3. Implementation Tasks

Tasks are ordered by dependency. Each task is a discrete, mergeable unit of work.

### Task 1: [Task Name]
- **Agent:** [database-expert | backend-expert | qa-expert]
- **Type:** [Schema | Service | Controller | Test | Config]
- **Owned Files:** (exclusive — only this agent may create/modify these)
  - `path/to/file.ts`
- **Files to create/modify:**
  - `path/to/file.ts` — [what to do]
  - `path/to/other.ts` — [what to do]
- **Details:**
  [Specific implementation instructions — what to build, how it should behave, what patterns to follow from the existing codebase]
- **Acceptance Criteria:**
  - [ ] [Criterion]
- **Dependencies:** None | Task N

### Task 2: [Task Name]
- **Agent:** [agent name]
- **Type:** [type]
- **Owned Files:**
  - `path/to/file.ts`
- **Files to create/modify:**
  - ...
- **Details:**
  [Specific instructions]
- **Acceptance Criteria:**
  - [ ] [Criterion]
- **Dependencies:** Task 1

[Continue for all tasks...]

---

## 4. Database Changes

### New Models / Tables
```
[Schema definition in the project's ORM syntax]
```

### Migrations Required
1. [Migration description]

### Data Backfill
[If applicable — or "None"]

---

## 5. Testing Requirements

### Unit Tests
| Test File | What to Test | Priority |
|-----------|-------------|----------|
| `path/to/test.spec.ts` | [what] | [High/Med/Low] |

### Integration Tests
| Test File | What to Test | Priority |
|-----------|-------------|----------|
| `path/to/test.spec.ts` | [what] | [High/Med/Low] |

### Edge Cases to Cover
- [Edge case 1]
- [Edge case 2]

### Test-First Specifications
These tests should be written BEFORE implementation (during Phase B), based on acceptance criteria and API contract. The qa-expert writes these in parallel with the implementation.

| Test File | Tests Against | Specification Source |
|-----------|--------------|---------------------|
| `path/to/test.spec.ts` | [service/controller method] | [acceptance criterion or API contract section] |

---

## 6. Configuration & Environment

### New Environment Variables
| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| [VAR_NAME] | Yes/No | [default] | [purpose] |

### New Dependencies
| Package | Version | Purpose |
|---------|---------|---------|
| [package] | [version] | [why] |

---

## 7. Rollout & Risk

### Risks
| Risk | Severity | Mitigation |
|------|----------|------------|
| [risk] | [High/Med/Low] | [mitigation] |

### Rollback Plan
[How to safely roll back if something goes wrong]

---

## 8. Agent Orchestration Order

This is the execution sequence for `/development implement`:

> **Mode selection:** If the feature has fewer than 5 implementation tasks or all tasks are linearly dependent, use **Sequential mode** (collapse Phase B into a single serial sequence). Otherwise, use **Parallel mode** with the Phase A/B/C structure below.
>
> **Mode: Sequential / Parallel** _(choose one)_

### Phase A: Foundation (sequential)
1. **database-expert** — [Task N: schema/migration work]
   - _Owned files: `src/database/migrations/`, `prisma/schema.prisma` (or equivalent)_
   - _After schema work: generate interface stubs at planned file paths so tests can import them_

### Phase B: Parallel Workstreams
These workstreams run concurrently. Each agent owns exclusive files — no overlap allowed.

| Workstream | Agent | Tasks | Owned Files |
|------------|-------|-------|-------------|
| test-spec | qa-expert | [Task N: write tests from specs] | `src/**/*.spec.ts`, `test/**/*` |
| implementation | backend-expert | [Task N: services, controllers, DTOs] | `src/modules/**/[feature].*` (excluding `*.spec.ts`) |

_Workstreams with no file overlap can be launched in parallel._
_If using Sequential mode, run these in order: implementation first, then test-spec._

### Phase C: Convergence (sequential)
3. **qa-expert** — [Task N: run tests, validate implementation, iterate until green]
   - _If tests fail → route failing tests + implementation to **backend-expert** for fixes_
   - _Instruction to fixing agent: "Make the test pass. Do not modify the test file."_
   - _Remediation loop: max 3 cycles_
4. **technical-lead** — [Final review of all changes]
   - _If blocking issues found → route fixes to responsible agent → re-review (max 2 cycles)_

### File Ownership Map
| File Pattern | Owner | Phase |
|-------------|-------|-------|
| `migrations/*`, `schema.*` | database-expert | A |
| `src/modules/**/[feature].service.ts` | backend-expert | B |
| `src/modules/**/[feature].controller.ts` | backend-expert | B |
| `src/modules/**/dto/*` | backend-expert | B |
| `**/*.spec.ts`, `test/**` | qa-expert | B |

---

## 9. Notes
[Any additional context, open questions, or follow-up items]
```

---

### File 2: `api-contract.md`

The handshake doc between backend and frontend — endpoint specs, request/response shapes, error formats.

```markdown
# API Contract: [Feature Name]

> Generated by `/development plan` on [date]

---

## Endpoints

### [METHOD] `/path`
- **Description:** [what this endpoint does]
- **Auth:** [auth type required]

**Request:**
```json
{
  "field": "type — description"
}
```

**Response (200):**
```json
{
  "field": "type — description"
}
```

**Error Responses:**
| Status | Code | Description |
|--------|------|-------------|
| 400 | `VALIDATION_ERROR` | [when this happens] |
| 401 | `UNAUTHORIZED` | [when this happens] |
| 404 | `NOT_FOUND` | [when this happens] |

[Repeat for each endpoint...]

---

## DTOs / Validation

### [DTOName]
```typescript
[DTO class/interface with validation decorators and field descriptions]
```

[Repeat for each DTO...]

---

## Common Error Format
```json
{
  "statusCode": 400,
  "message": "Human-readable error message",
  "error": "ERROR_CODE"
}
```

---

## Notes
[Any API-specific notes: rate limits, pagination format, versioning, etc.]
```

---

### File 3: `ui-consumption-guide.md`

How the frontend should integrate with this feature.

```markdown
# UI Consumption Guide: [Feature Name]

> Generated by `/development plan` on [date]

---

## Overview
[What this feature looks like from the UI perspective — what the user sees and does]

---

## User Flows

### Flow 1: [Flow Name]
```
[Step-by-step user flow]
1. User does X
2. UI calls [endpoint]
3. UI displays [response]
4. ...
```

[Repeat for each flow...]

---

## API Integration

### Endpoints to Consume
| Endpoint | When to Call | Request | Key Response Fields |
|----------|-------------|---------|-------------------|
| `[METHOD] /path` | [trigger] | [key params] | [fields the UI needs] |

### Authentication
[How to pass auth tokens, any special headers required]

---

## State Management Recommendations
[Suggestions for how to manage this feature's state — caching, optimistic updates, polling vs websockets, etc.]

---

## Loading & Error States
| State | Trigger | Suggested UX |
|-------|---------|-------------|
| Loading | [when] | [spinner, skeleton, etc.] |
| Empty | [when] | [empty state message/CTA] |
| Error | [when] | [error message, retry button, etc.] |

---

## Edge Cases & UX Considerations
- [Edge case 1 — how UI should handle it]
- [Edge case 2 — how UI should handle it]

---

## Notes
[Any additional context for frontend developers]
```

---

### File 4: `test-report.md`

QA audit trail — populated during `/development implement`.

```markdown
# Test Report: [Feature Name]

> Created by `/development plan` on [date]
> Last updated: [date]

---

## Test Summary
| Metric | Value |
|--------|-------|
| Total Tests | — |
| Passing | — |
| Failing | — |
| Coverage | — |

---

## Test Results

_Populated during `/development implement`_

### Unit Tests
| Test | File | Status | Notes |
|------|------|--------|-------|
| — | — | — | — |

### Integration Tests
| Test | File | Status | Notes |
|------|------|--------|-------|
| — | — | — | — |

---

## Remediation Log

_Populated during `/development implement` if issues are found_

### Cycle 1
| Issue | Severity | Agent | Fix | Result |
|-------|----------|-------|-----|--------|
| — | — | — | — | — |

[Additional cycles if needed...]

---

## Notes
[Any QA-specific observations, known limitations, or manual testing notes]
```

---

### File 5: `progress-log.md`

Task checklist and status tracker — updated as implementation proceeds.

```markdown
# Progress Log: [Feature Name]

> Created by `/development plan` on [date]
> Last updated: [date]

---

## Status: **Not Started**

<!-- Statuses: Not Started | In Progress | QA Validation | Review | Implemented | Blocked -->

---

## Task Checklist

- [ ] **Task 1:** [Task name] — _Agent: [agent]_
- [ ] **Task 2:** [Task name] — _Agent: [agent]_
- [ ] **Task 3:** [Task name] — _Agent: [agent]_
[Continue for all tasks from implementation guide...]

---

## QA & Review

- [ ] QA validation passed
- [ ] Technical lead review passed
- [ ] All acceptance criteria met

## Documentation (optional — via `/development document`)

- [ ] Feature flow documented (`feature-flow.md`)
- [ ] Swagger decorators verified
- [ ] API contract synced with implementation
- [ ] UI consumption guide synced with implementation
- [ ] `.env.example` updated
- [ ] Error catalog updated
- [ ] CLAUDE.md updated
- [ ] Knowledge base refreshed

---

## Activity Log

| Date | Agent | Action | Details |
|------|-------|--------|---------|
| — | — | — | _Populated during implementation_ |

---

## Blockers
[Any current blockers — or "None"]
```

---

### File 6: `decisions.md`

ADR-lite — records key decisions made during planning and implementation.

```markdown
# Decisions: [Feature Name]

> Created by `/development plan` on [date]

---

## Planning Decisions

| # | Decision | Choice | Alternatives Considered | Rationale |
|---|----------|--------|------------------------|-----------|
| 1 | [decision point] | [chosen approach] | [what else was considered] | [why this was chosen] |

---

## Implementation Decisions

_Populated during `/development implement` if deviations from the plan occur_

| # | Decision | Choice | Original Plan | Reason for Change |
|---|----------|--------|--------------|-------------------|
| — | — | — | — | — |

---

## Notes
[Any additional decision context or open questions]
```

---

After writing all 6 files, confirm the folder path and summarize the implementation plan.
