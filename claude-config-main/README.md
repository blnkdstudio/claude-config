# Claude Config — Claude Code Plugin

Development workflow toolkit with expert agents, multi-agent orchestration, progress reporting, and a portable cross-project knowledge base.

## Install

On any machine with Claude Code:

```bash
# Add the marketplace (one-time)
/plugin marketplace add junreytejas/claude-config

# Install the plugin
/plugin install claude-config@junreytejas
```

All agents and skills are available immediately, namespaced as `/claude-config:*`.

### Update

```bash
/plugin marketplace update junreytejas
```

### Test locally

```bash
claude --plugin-dir /path/to/claude-config
```

## Skills

| Skill | Description |
|-------|-------------|
| `/claude-config:development build` | Start a new project from scratch (interactive discovery → scaffold → GitHub) |
| `/claude-config:development init` | Initialize project knowledge base (full codebase analysis) |
| `/claude-config:development plan` | Interactive planning session — generates feature folder |
| `/claude-config:development implement <path>` | Multi-agent orchestration to build a planned feature |
| `/claude-config:development resume <path>` | Diagnose and resume an interrupted implementation |
| `/claude-config:development document <path>` | Post-implementation documentation suite |
| `/claude-config:development update` | Refresh the knowledge base with recent changes |
| `/claude-config:development status` | Check knowledge base staleness |
| `/claude-config:development list` | Dashboard of all features and their status |
| `/claude-config:development ship [msg]` | Lint + build + test, then commit & push |
| `/claude-config:development research` | Investigate a topic — produces structured research brief |
| `/claude-config:development debug` | Multi-agent bug diagnosis, fix, and verification |
| `/claude-config:learn [description]` | Capture a cross-project learning to the portable knowledge base |
| `/claude-config:review-learnings [list\|prune\|consolidate]` | Review, prune, or consolidate accumulated learnings |

## Agents

| Agent | Model | Description |
|-------|-------|-------------|
| cto | sonnet | System architecture, technology strategy, security posture |
| technical-lead | sonnet | Code review, architecture decisions, task breakdown |
| backend-expert | sonnet | API design, service architecture, integration patterns |
| database-expert | sonnet | Schema design, query optimization, migrations |
| qa-expert | sonnet | Test strategy, writing tests, bug hunting |
| process-cartographer | sonnet | End-to-end process tracing and documentation |
| daily-report | opus | Daily progress report from git history |
| weekly-report | opus | Weekly progress report from git history |
| monthly-report | opus | Monthly progress report from git history |

## Hooks

| Event | Trigger | Action |
|-------|---------|--------|
| PostToolUse | Edit/Write on `.ts`/`.tsx` files | Runs `tsc --noEmit` to type-check |

## Structure

```
claude-config/
├── .claude-plugin/
│   ├── plugin.json          # Plugin manifest
│   └── marketplace.json     # Marketplace catalog
├── agents/                  # 9 custom sub-agents
├── knowledge/               # Cross-project knowledge base (portable via git)
│   ├── KNOWLEDGE.md         # Always-loaded index
│   ├── patterns/            # Workflow preferences, coding conventions
│   ├── decisions/           # Architectural choices with rationale
│   └── playbooks/           # Step-by-step solutions to solved problems
├── skills/                  # 15 slash command skills
├── hooks/
│   └── hooks.json           # TypeScript type-check hook
└── CLAUDE.md                # Plugin rules
```

## Knowledge Base

A portable, curated knowledge base that accumulates cross-project learnings over time and travels with the plugin via git.

```
knowledge/
├── KNOWLEDGE.md           # Always-loaded index
├── patterns/              # Workflow preferences, coding conventions
├── decisions/             # Architectural choices with rationale
└── playbooks/             # Step-by-step solutions to solved problems
```

**How it works:**
- During normal conversation, Claude watches for reusable insights (corrections, decisions, solved problems)
- When detected, Claude proposes capturing it — nothing is saved without your approval
- Use `/claude-config:learn` to explicitly capture a learning
- Use `/claude-config:review-learnings` to audit, prune, or consolidate over time
- Commit `knowledge/` to git to sync across machines
