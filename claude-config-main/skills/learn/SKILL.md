---
description: "Capture a cross-project learning (pattern, decision, or playbook) into the portable knowledge base. Use when Claude detects a reusable insight during conversation, or when the user wants to explicitly save a learning. Examples: 'learn always validate external API payloads before building features', 'learn we chose BullMQ over Agenda because of Redis reuse'."
---

# /claude-config:learn

You are capturing a cross-project learning into the portable knowledge base at `knowledge/` in this plugin's root directory. This knowledge travels with the user across machines and projects via git.

The user ran: `/claude-config:learn $ARGUMENTS`

---

## Step 1: Parse Input

**If `$ARGUMENTS` is provided:**
- Use it as the raw learning description. Proceed to Step 2.

**If no arguments:**
- Ask the user: "What did you learn? Describe the insight, pattern, decision, or solution you want to capture."
- Wait for their response before proceeding.

---

## Step 2: Classify the Learning

Determine the type based on the content:

| Type | Signal | Example |
|------|--------|---------|
| **pattern** | A preference, convention, or confirmed approach that should be repeated | "Always use `prisma db push` instead of `prisma migrate dev`" |
| **decision** | A deliberate choice between alternatives with rationale | "Chose BullMQ over Agenda because we already run Redis" |
| **playbook** | A step-by-step solution to a non-obvious problem | "To fix MariaDB JSON escaping, use Prisma seed scripts instead of raw SQL" |

Present your classification:

> I'd classify this as a **[type]**. Does that sound right, or should it be a different type?

Wait for confirmation. If the user corrects the type, use their choice.

---

## Step 3: Check for Duplicates

1. Read `knowledge/KNOWLEDGE.md` (the index file, relative to this plugin's root directory).
2. Scan all existing entries — compare titles and descriptions for semantic overlap with the new learning.
3. If a related entry exists, present it:

> An existing entry covers similar ground:
> - **[Title]** — [description]
>
> Should I: **(a)** update the existing one, **(b)** create a new entry anyway, or **(c)** skip?

- If **(a)**: Read the existing file, merge the new insight, and proceed to Step 4 with the updated content.
- If **(b)**: Proceed to Step 4 as a new entry.
- If **(c)**: Stop. Respond: "Skipped. No changes made."

If no duplicates found, proceed to Step 4.

---

## Step 4: Generate the Knowledge File

Create the full file content based on the type:

### For `pattern`:

```markdown
---
name: "[Concise title in imperative form]"
description: "[One-line description for relevance matching]"
type: pattern
tags: [tag1, tag2, tag3]
created: "[today's date, YYYY-MM-DD]"
source: "[name of current project, or 'general' if not project-specific]"
---

[Rule statement — what to do or not do]

**Why:** [The reason this matters — ideally referencing a past incident or strong preference]

**How to apply:** [When and where this guidance kicks in]
```

### For `decision`:

```markdown
---
name: "[Decision title]"
description: "[One-line description for relevance matching]"
type: decision
tags: [tag1, tag2, tag3]
created: "[today's date, YYYY-MM-DD]"
source: "[name of current project, or 'general' if not project-specific]"
---

[Decision statement — what was chosen]

**Context:** [What prompted this decision]

**Alternatives considered:** [What else was evaluated]

**Why this choice:** [The rationale]

**When to reconsider:** [Conditions that would make this decision worth revisiting]
```

### For `playbook`:

```markdown
---
name: "[Problem title]"
description: "[One-line description for relevance matching]"
type: playbook
tags: [tag1, tag2, tag3]
created: "[today's date, YYYY-MM-DD]"
source: "[name of current project, or 'general' if not project-specific]"
---

[Problem statement — what goes wrong or what you need to accomplish]

## Steps

1. [First step]
2. [Second step]
3. [Continue as needed]

**Gotchas:** [Non-obvious pitfalls or things that will trip you up]
```

### Tag guidelines:
- Use lowercase, hyphenated tags
- Include technology tags (e.g., `nestjs`, `prisma`, `react`, `docker`)
- Include domain tags (e.g., `deployment`, `testing`, `api-design`, `performance`)
- 2-5 tags per entry

### File naming:
- Use the type as prefix: `pattern_`, `decision_`, `playbook_`
- Use kebab-case for the rest: `pattern_validate-external-apis.md`
- Keep names short but descriptive

---

## Step 5: Present for Approval

Show the complete file content in a fenced code block:

> Here's the knowledge file I'd save:
>
> ```markdown
> [full file content]
> ```
>
> Save to `knowledge/[type]s/[filename].md`? **(yes / edit / no)**

- **yes**: Proceed to Step 6.
- **edit**: Ask what to change, apply edits, re-present.
- **no**: Stop. Respond: "Discarded. No changes made."

---

## Step 6: Write and Index

1. **Write the file** to `knowledge/[type]s/[filename].md` (relative to this plugin's root directory).

2. **Update the index** — read `knowledge/KNOWLEDGE.md`, then add a new entry line under the appropriate section heading:
   - `## Patterns` for patterns
   - `## Decisions` for decisions
   - `## Playbooks` for playbooks

   Entry format (under 150 characters):
   ```
   - [Title](types/filename.md) — one-line hook
   ```

3. **Confirm:**

> Saved to `knowledge/[type]s/[filename].md` and indexed in KNOWLEDGE.md.
>
> To persist across machines, commit the `knowledge/` directory:
> ```bash
> cd /path/to/claude-config && git add knowledge/ && git commit -m "learn: [short description]"
> ```
