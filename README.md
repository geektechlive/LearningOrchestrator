 LearningOrchestra README 
 
 — Quick Reference Card

## How It Works (30-second version)

1. You ask Claude to do something
2. Claude scores the complexity (0-12 scale)
3. Score 0-2 → does it directly (SINGLE)
4. Score 3-5 → plans first, maybe 1-2 helpers (GUIDED)
5. Score 6+ → full decomposition with parallel workers (ORCHESTRA)

## The Agents

**Core:**
```
orchestrator-planner (Opus, high effort)   → Breaks big tasks into small ones
code-worker (Sonnet, medium effort)        → Generic code — writes/modifies code
research-worker (Sonnet, medium effort)    → Investigates, compares, analyzes (read-only)
project-worker (Haiku, low effort)         → Moves files, scaffolds, organizes
validator (Haiku, low effort)              → Runs lint/build/test checks
code-reviewer (Sonnet, medium effort)      → Reviews code quality (read-only)
```

**Stack-specific (auto-selected by planner):**
```
astro-supabase-worker (Sonnet, medium)     → Astro + Supabase sites
swiftui-worker (Sonnet, medium)            → SwiftUI / SwiftData / MV architecture
react-worker (Sonnet, medium)              → React apps (non-Next.js)
nextjs-worker (Sonnet, medium)             → Next.js App Router
```

## Override Commands

| Command | Effect |
|---------|--------|
| "Just do it" | Skip orchestration, single mode |
| "Full orchestra" | Force full orchestration |
| "Plan this but don't execute" | See the plan before committing |

## Task Types

| Type | Detected By | Workers Get |
|------|-------------|-------------|
| CODE | Files, functions, builds, bugs | Full read/write/bash |
| PROJECT | Organize, scaffold, restructure | Full read/write/bash |
| RESEARCH | Compare, analyze, recommend | Read-only + web |
| HYBRID | Multi-phase (research → build) | Phased access |

## Model Tiers

| Tier | Model | Used For | Cost |
|------|-------|----------|------|
| T1 | Opus | Planning, architecture, ambiguity | $$$ |
| T2 | Sonnet | Code, research, review | $$ |
| T3 | Haiku | Validation, file ops, boilerplate | $ |

## Workspace

```
.orchestra/
├── manifest.md      ← What's planned
├── results/         ← What workers produced
├── validation.md    ← Quality check results
└── summary.md       ← Final consolidated report
```

## Safety Limits
- Max 6 concurrent workers
- Destructive ops require approval
- 3+ Opus workers triggers usage warning
- Workers can't read each other's output

## Context Economy (Pro Plan Optimization)
- Grep/Glob before Read — never open full files to find something
- Scoped reads only — read the lines you need, not the whole file
- Write once — plan before coding, don't write-delete-rewrite
- Concise reports — summarize changes, don't echo file contents
- Batch small tasks — 3 changes to 1 file = 1 worker, not 3

## Thinking Token Awareness
- Opus planner burns ~3x more allocation than Sonnet due to thinking tokens
- Planner marks "Straightforward decomposition" when deep reasoning isn't needed
- Haiku workers at low effort = minimal thinking overhead
- Overage credits cover rate limit resets — Orchestra completes uninterrupted

## Post-Execution Retrospective (MANDATORY)
After every Orchestra run, the orchestrator MUST:
1. Write a retrospective in `.orchestra/summary.md`
2. Evaluate scoring accuracy, tier accuracy, effort accuracy, context economy
3. Propose adjustments if needed → ask user to approve before changing
4. Track patterns across runs → proactively flag recurring issues

## Stack Detection
The planner auto-detects project stacks and selects the right agent:
- `astro.config.*` + Supabase → `astro-supabase-worker`
- `.xcodeproj` or SwiftUI imports → `swiftui-worker`
- `next.config.*` or `app/layout.tsx` → `nextjs-worker`
- `react` in package.json (no Next.js) → `react-worker`
- No match → falls back to generic `code-worker`
