---
title: Agent Orchestration Patterns
type: pattern
created: 2026-05-15
updated: 2026-05-15
---

# Agent Orchestration Patterns

## Parallel Agent Rules
- Never assign two agents to modify the same file
- Group tasks that share imports/module wiring into one agent
- Each agent must READ files before modifying (Edit, not Write for existing files)
- Agents must run build commands to verify before reporting done
- Max 3 worker agents per cycle

## Agent Specialization
- **CTO agents** (2 parallel): internal review + market research. Read-only on code, write to backlog/
- **Tech Lead** (1 sequential): reads backlog + code, writes tasks/. Must check completed/ to avoid re-doing work
- **Workers** (2-3 parallel): read tasks/, modify source code, build, verify
- **QA agent** (post-workers): verify builds pass, check integration points match, run type checks

## File-Based Communication
- Agents communicate ONLY through markdown files (docs/rnd/)
- Each file has a standard format with frontmatter
- Numbering: backlog 001+, tasks T001+, cycles cycle-001+
- Agents must check existing numbers before creating new files

## Convention Enforcement
- Agents must follow project's existing naming patterns (read existing code first)
- API path conventions must be consistent — grep to verify after changes
- Mock/stub implementations must mirror real implementations (type systems catch mismatches)
- Never return sensitive fields in API responses

## Session & Context Management
- Run /compact after each R&D cycle to free context
- All persistent state lives in files (docs/rnd/, memory/, knowledge/)
- Nothing important should be stored only in conversation context
- Prefer local /compact over remote triggers (simpler, fewer auth dependencies)

## Deployment Verification
- After code changes: verify build passes for all affected projects
- After deploy: check container status, read logs for startup errors
- Verify the deployed artifact actually contains the new code (check bundle hash, grep for new functions)

## Common Pitfalls
1. **Integration mismatches** — frontend calls one path, backend expects another. Grep to verify.
2. **Build cache** — Docker/bundler caches can serve stale code. Use --no-cache in CI/CD.
3. **Container conflicts** — orphan containers block recreate. Always `down` before `up`.
4. **Special chars in credentials** — symbols in passwords break URL encoding. Use alphanumeric.
5. **Missing mock updates** — adding methods to real services without updating mocks breaks type checks.
6. **Stale deploys** — verify the deployed artifact changed, not just that the container restarted.
7. **Schema drift** — after DB schema changes, migration/push must run before app starts.
