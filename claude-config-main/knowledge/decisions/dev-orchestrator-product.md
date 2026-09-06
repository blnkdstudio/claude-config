---
title: "Fission — Product Development Orchestrator"
type: decision
created: 2026-05-15
tags: [product-idea, meta, orchestration, fission]
---

# Fission — Product Development Orchestrator

**Tagline:** One idea. Chain reaction.

## Concept
A web-based platform that manages multiple projects (GitHub repos), each running its own autonomous R&D pipeline (CTO → Tech Lead → Workers). A solo developer or small agency manages a portfolio of continuously-improving products.

The nuclear fission metaphor: one idea (neutron) enters the reactor → splits into findings → findings split into tasks → tasks become code → code reveals more findings → self-sustaining chain reaction.

## Origin
Built organically while developing Exybit ISP. The R&D pipeline evolved through three generations in a single session:
1. Manual cycles with /compact between them (user blocked)
2. Agents-as-compaction (no manual /compact needed)
3. Continuous pipeline (no cycles, filesystem-as-queue, user never blocked)

The realization: if this works for one project, it works for N projects simultaneously.

## Architecture Sketch
```
Fission Dashboard
├── Projects (each = GitHub repo + persistent pipeline)
│   ├── Reactor view (kanban: backlog → tasks → in-progress → done)
│   ├── Agent status (running agents, current work)
│   ├── Steering controls ("focus on security", "pause", "ship it")
│   └── Activity feed (commits, findings, completions)
├── Knowledge Core (cross-project learnings, enriches with every reaction)
└── Settings (model tiers, token budgets, GitHub connections)
```

## Key Building Blocks (all exist today)
- Claude API — spawns agent sessions programmatically
- GitHub API — repo management, commits, PRs
- R&D Pipeline playbook — the workflow logic (already documented in claude-config)
- Filesystem-as-queue → becomes database tables (backlog, tasks, in-progress, completed)
- Knowledge system — shared across projects, compounds with usage (the fuel rods)

## Nuclear Metaphor Mapping
| Nuclear | Fission Platform |
|---------|-----------------|
| Neutron | User's idea / initial input |
| Fission event | CTO review splits idea into findings |
| Released neutrons | Each finding becomes new tasks |
| Chain reaction | Tasks → code → new findings → more tasks |
| Critical mass | Pipeline becomes self-sustaining |
| Fuel rods | Knowledge system (enriches over time) |
| Reactor | The project's pipeline environment |
| Control rods | User steering ("pause", "focus on X", "stop") |

## Business Model
- Solo dev / small agency runs 5-10 projects continuously
- User becomes a product portfolio manager, not a coder
- Wake up → review PRs → approve/steer → projects improve overnight
- Cross-project knowledge means each new project starts smarter

## Architecture Decision: Dashboard-First (no raw terminal)

### Core Insight
Claude Code CLI is the engine. Fission is a dashboard + pipeline runner. No web terminal needed — the backend spawns `claude -p` commands headlessly. The user interacts through a kanban board and input box, not a terminal.

### How It Works
1. User navigates to `project.com/fission` (hidden route, like /wp-admin)
2. Authenticates (JWT)
3. Sees: kanban board (Trello-like) + pipeline controls + input box
4. Clicks "Run Pipeline" → backend spawns `claude -p "run rnd"` headlessly
5. Pipeline state syncs from docs/rnd/ files → DB → kanban updates in real-time
6. User types directions in input box → "Add a billing export feature" or "Focus on security"
   → backend spawns `claude -p "User wants: {input}. Read docs/rnd/ and act."

### Frontend UX
```
/fission Dashboard
├── Pipeline Button → "Run Pipeline"
│   └── Backend: claude -p "run rnd" --output-format json
│   └── Kanban updates in real-time as pipeline progresses
├── Kanban Board (Trello-like drag-and-drop)
│   ├── Backlog (findings from CTO agents)
│   ├── Planned (tasks from Tech Lead)
│   ├── In Progress (workers building)
│   └── Completed (done)
├── Direction Input → free-text box for user steering
│   └── "Add billing export" → creates finding in backlog
│   └── "Focus on security" → steers next pipeline run priority
│   └── Backend translates to claude -p prompts
└── Project Stats (findings count, tasks done, recent commits)
```

### Backend Architecture
```
NestJS Backend
├── REST API (projects, findings, tasks, dashboard)
├── Pipeline Runner (spawns claude -p commands via node-pty)
├── File Watcher (syncs docs/rnd/ → DB for kanban)
└── DB (PostgreSQL — read cache of pipeline state)
```

### AI Cost Strategy
```
Phase 1 (bootstrap):  Claude Code CLI on server — $0 extra (subscription)
Phase 2 (revenue):    Paying users → revenue covers Claude API costs
Phase 3 (scale):      Claude API direct + custom sandbox (the v0 pattern)
```

Pluggable executor interface allows swapping CLI → API without changing the rest:
```typescript
interface AgentExecutor {
  run(prompt: string, repoPath: string): Promise<AgentResult>;
}
class CliExecutor implements AgentExecutor { /* shells out to claude -p */ }
class ApiExecutor implements AgentExecutor { /* calls Claude Messages API */ }
```

## Distribution Strategy: Self-Hosted (WordPress.org model)

Fission ships as a **Docker image**. Clients deploy on their own VPS alongside their projects. No central Fission server needed.

### Client Setup
```yaml
# Client adds to their docker-compose.yml
services:
  fission:
    image: junreytejas/fission
    ports:
      - "3001:3001"
    volumes:
      - /path/to/project:/workspace
    environment:
      - DATABASE_URL=postgresql://...
```

Client brings their own:
- VPS (where their project already runs)
- Claude subscription (Claude Code installed + logged in)
- Fission just adds the dashboard layer

### Revenue Models
| Model | How |
|-------|-----|
| Open core | Free self-hosted, paid premium features (team collab, analytics) |
| Managed hosting | Host Fission for clients who don't want to self-manage |
| License | Annual license fee per server |
| Support | Paid support/onboarding packages |

### Why Self-Hosted Wins
- Zero recurring infrastructure cost for Fission (client pays their own hosting)
- Client data stays on their server (privacy/compliance)
- Same codebase whether self-hosted or managed
- Lower barrier to adoption (no SaaS signup, just docker pull)

## Meta Property
Fission will use its own R&D pipeline to build itself. First project in the reactor is Fission.

## Repo
GitHub: junreytejas/Fission
Symlinks claude-config for shared knowledge from day one.
