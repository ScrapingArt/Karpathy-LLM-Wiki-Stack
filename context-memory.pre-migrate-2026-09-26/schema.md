# Memory Schema

This file documents the structure of all memory files used by the agent stack. Read this to understand what each file contains and when to consult it.

## Global Memory (`~/.agents/memory/`)

| File | Purpose | Read by |
|------|---------|---------|
| `style.md` | Language-agnostic code style rules + per-language overrides | Developer, Reviewer |
| `conventions.md` | Commit format, branch naming, PR structure, CHANGELOG format | Developer, Documenter |
| `review-criteria.md` | Adversarial review checklist + findings format | Reviewer |
| `schema.md` | This file — memory structure reference | All agents |

## Per-Project Context (`.context/`)

| File | Purpose | Created by | Read by |
|------|---------|-----------|---------|
| `CONTEXT.md` | Tech stack, architecture constraints, key files, off-limits | `agents init` | All agents |
| `PLAN.md` | Current feature plan (phases, tasks, acceptance criteria) | Dispatcher | Developer, Reviewer |
| `MAP.md` | Categorized file map of the codebase | Explorer | Developer, Documenter |

## PLAN.md Schema

```markdown
---
feature: <name>
created: <ISO date>
status: draft | approved | in-progress | complete
current_phase: 1
total_phases: N
---

# Plan: <Feature Name>

## Objective
One sentence.

## Scope
Files in scope:
- path/to/file — reason

Files out of scope:
- path/to/other — reason

## Phase 1: <Name>
### Tasks
- [ ] Task 1
- [ ] Task 2
### Acceptance Criteria
- [ ] Criterion A
```

## MAP.md Schema

```markdown
---
generated: <ISO date>
scope: <description of what was searched>
---

# Codebase Map

## Entry Points
- path/to/entry — description

## Domain: <Name>
- path/to/file — description

## Tests
- path/to/test — what it covers

## Config / Infra
- path/to/config — what it controls
```
