# CLAUDE.md

Claude Code plugin distributed via marketplace at `junreytejas/claude-config`.

## Structure

```
claude-config/
├── .claude-plugin/
│   ├── plugin.json          # Plugin identity (name, version, author)
│   └── marketplace.json     # Marketplace catalog entry
├── agents/                  # 9 sub-agents (YAML frontmatter)
│   ├── backend-expert.md    # API design, service architecture (sonnet)
│   ├── cto.md               # System architecture, tech strategy (sonnet)
│   ├── daily-report.md      # Daily git progress report (opus)
│   ├── database-expert.md   # Schema design, query optimization (sonnet)
│   ├── monthly-report.md    # Monthly git progress report (opus)
│   ├── process-cartographer.md # End-to-end process tracing (sonnet)
│   ├── qa-expert.md         # Test strategy, bug hunting (sonnet)
│   ├── technical-lead.md    # Code review, task breakdown (sonnet)
│   └── weekly-report.md     # Weekly git progress report (opus)
├── knowledge/               # Cross-project learning layer (portable via git)
│   ├── KNOWLEDGE.md         # Always-loaded index
│   ├── patterns/            # Workflow preferences, coding conventions
│   ├── decisions/           # Architectural choices with rationale
│   └── playbooks/           # Step-by-step solutions to solved problems
├── skills/                  # 15 slash commands (SKILL.md format)
│   ├── development/         # Orchestrator — routes to subcommands
│   ├── development-build/   # Start new project from scratch (discovery → scaffold → GitHub)
│   ├── development-init/    # Initialize project knowledge base
│   ├── development-plan/    # Interactive feature planning session
│   ├── development-implement/ # Multi-agent feature build
│   ├── development-resume/  # Resume interrupted implementation
│   ├── development-document/ # Post-implementation docs
│   ├── development-update/  # Refresh knowledge base
│   ├── development-status/  # Check knowledge base staleness
│   ├── development-list/    # Feature status dashboard
│   ├── development-ship/    # Lint + build + test, commit & push
│   ├── development-research/ # Investigate a topic, produce research brief
│   ├── development-debug/   # Multi-agent bug diagnosis and fix
│   ├── learn/               # Capture a cross-project learning
│   └── review-learnings/    # Review, prune, consolidate learnings
├── hooks/
│   └── hooks.json           # PostToolUse: tsc --noEmit on .ts/.tsx edits
└── CLAUDE.md
```

## Rules

- Agent files must have YAML frontmatter with `name`, `description`, `model`, and `color`
- Skill files use `skills/<name>/SKILL.md` format with `description` frontmatter
- Plugin manifest in `.claude-plugin/plugin.json` defines the plugin identity
- Marketplace catalog in `.claude-plugin/marketplace.json` must match plugin.json metadata
- Skills are namespaced as `/claude-config:*` when installed
- Test plugin changes locally: `claude --plugin-dir .`, then verify skills and agents load

## Global Development Rules

These rules are enforced by this plugin across all projects where it is installed.

### Dead Code Verification

Before removing any function, class, method, import, file, or code block:

1. Search the entire codebase for all references to the symbol (use Grep with the exact name)
2. Check for dynamic references (string-based lookups, reflection, decorator metadata)
3. Check for references in test files, configuration files, and scripts
4. Only remove the code after confirming zero active references

If there is any doubt, ask the user before deleting. Never assume code is unused based on a single file's context alone.

### File Overwrite Protection

Before writing or overwriting any existing file:

1. Read the current file contents first
2. Understand what already exists in the file
3. Preserve existing functionality, imports, and patterns unless explicitly asked to remove them
4. When adding to a file, merge new code with existing code rather than replacing the file wholesale

Never use the Write tool on an existing file without first using the Read tool on that same file in the current conversation. If you have not read the file in this session, read it before editing.

### Deployment Safety

1. **Never use `prisma migrate dev`** — it wipes the database. Use `prisma db push` for schema sync or manual SQL for targeted changes.
2. **Seed data with Prisma scripts** (`npx tsx prisma/seed-*.ts`), not raw SQL — MariaDB has strict JSON validation that makes SQL INSERT with JSON painful due to quote escaping.
3. **Always type-check before pushing** — run `tsc --noEmit` or verify no unused/missing imports in changed files. Docker builds will fail on TypeScript errors that dev mode tolerates.
4. **`git pull` does not rebuild Docker containers** — always `docker compose build --no-cache && docker compose up -d --force-recreate` after pulling new code on servers.
5. **Non-TypeScript assets** (.hbs, .html, static files) are not copied by `nest build` — add explicit `cp -r` commands in the Dockerfile after the build step. Do not rely on `nest-cli.json` assets config when output nests under `dist/src/`.

### External API Integration

1. **Test external API payloads early** — before building features around an assumed contract, send a real test request to verify what the API actually accepts. Documentation may be incomplete or wrong.
2. **Validate API response structure** — check that the data you send is reflected correctly in the response before building the full UI/backend around it.

### NestJS Patterns

1. **Route order matters** — static routes (`/variables`, `/companies`, `/users`) must be declared before parameterized routes (`/:id`) in controllers, or NestJS will match the literal string as a param and fail validation.
2. **BigInt serialization** — `JSON.stringify` cannot serialize BigInt. Use a replacer function: `JSON.stringify(data, (_k, v) => typeof v === 'bigint' ? v.toString() : v)` when caching Prisma results in Redis.
3. **ThrottleTier values** — only `'default'`, `'fire-and-forget'`, and `'heavy'` are valid. Do not use `'standard'` or other arbitrary values.

## R&D Pipeline

When the user says **"run rnd"**, start the autonomous R&D pipeline:

1. Read `knowledge/playbooks/rnd-autonomous-loop.md` for the full playbook
2. Create `docs/rnd/` folder structure if it doesn't exist (backlog/, tasks/, in-progress/, completed/, log/)
3. Create `docs/rnd/state.md` if it doesn't exist (scan the codebase and write a ~50 line project summary)
4. Execute the pipeline: launch CTO agents (background) → Tech Lead → Workers → Wrap-up
5. Follow the playbook's principles: agents-as-compaction, continuous pipeline, model tiering, lean prompts
6. Continue until user says "stop" or "halt"

The pipeline runs autonomously. The user can chat, review, or steer while agents work in the background.

## Continuous Learning

This plugin maintains a portable, curated knowledge base that accumulates cross-project learnings over time. It travels with the user across machines and projects via git.

### Cross-Project Knowledge Base

The knowledge base lives at `knowledge/KNOWLEDGE.md` (relative to this plugin's root directory). **Read this index at the start of every conversation.** When a task relates to a topic covered by an indexed knowledge file (match by tags or description), read that file and apply its guidance.

- **Patterns** — confirmed workflow preferences and coding conventions. Apply unless context clearly differs.
- **Decisions** — architectural choices with rationale. Apply unless the current project has different constraints.
- **Playbooks** — proven step-by-step solutions to recurring problems. Follow when the same problem arises.

This knowledge is cross-project. It applies everywhere this plugin is installed.

### Learning Detection

Watch for these signals during conversation. When you detect one, suggest capturing it with `/claude-config:learn`. **Do NOT auto-save.** Only suggest when the signal is strong and the insight is cross-project (not specific to the current project's schema, data, or config).

**Signals to watch for:**

1. **Correction that reveals a preference** — user corrects your approach and the correction would apply in other projects too. ("Don't use mocks for database tests" → pattern)
2. **Architectural decision with rationale** — user makes a deliberate choice between alternatives and explains why. ("We chose BullMQ over Agenda because..." → decision)
3. **Solved problem that was non-obvious** — a debugging session or implementation reveals a solution that would be hard to rediscover. ("The fix for MariaDB JSON escaping is..." → playbook)
4. **Confirmed approach** — user explicitly validates a non-obvious approach you chose. ("Yes, exactly — always do it that way" → pattern)

**When NOT to suggest learning capture:**

- Trivial corrections (typos, wrong file name)
- Project-specific facts (this project's schema, this project's deployment URL)
- Information already in the knowledge base (check the index first)
- Information already in the project's per-project memory
- Obvious best practices that any developer would know

**Suggestion format:**

> This seems like a reusable [pattern/decision/playbook]. Want me to capture it with `/claude-config:learn`?

### Knowledge Base vs. Project Memory

| Scope | Where | Example |
|-------|-------|---------|
| Cross-project (applies everywhere) | `knowledge/` in this plugin | "Always validate external API payloads before building features around them" |
| Project-specific (applies here only) | `~/.claude/projects/{path}/memory/` | "This project uses MariaDB 10.6 with Prisma 6" |

**Rule of thumb:** If the insight would be useful in a different project with a different tech stack, it belongs in `knowledge/`. If it depends on this project's specific setup, it belongs in project memory.
