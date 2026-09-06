---
title: Fission UX Principles
type: pattern
created: 2026-05-15
tags: [fission, ux, design]
---

# Fission UX Principles

## Information Hierarchy (top to bottom)
1. **Project name + live status badge** — one glance answers "is the AI running?"
2. **Currently working on** — names the actual task and file. The missing piece that connects status to work.
3. **Queue summary** — three counts in one line. Useful but secondary.
4. **Kanban board** — the work itself. In Progress column highlighted, active card gets "AI WORKING" badge.
5. **Direct the AI** — accessible but visually quiet. A tool you reach for occasionally.
6. **Live activity log** — collapsed by default. Developers expand it; clients never need to.

## Two-Audience Design
- **Non-technical client** sees: project name, "AI working on X — 4m 11s," kanban with one card glowing. Done.
- **Developer** sees the same top section plus queue counts, direction input, and expandable log. Information is there, not shouting.

## Principles
- Stop button is contextual (on the current task, not standalone)
- Queue counts replace full panels — useful info doesn't need a card
- "AI WORKING" badge connects the status pill to the actual kanban card
- Collapsed log is a bet that most users never need it
- If AI works in parallel, "Currently working on" becomes a list

## Iteration Notes
- AI WORKING badge could pulse subtly (avoided getting fancy for MVP)
- Parallel tasks: "Currently working on (2 tasks)" with a list
- If developers rely on the log constantly, it should default open
