# LearningOrchestra — Installation & Setup Guide

## What You're Installing

Orchestra is a set of CLAUDE.md rules and subagent definitions that turn Claude Code into
an intelligent orchestration system. When you ask Claude Code to do something, Orchestra:

1. Assesses the task complexity against a weighted scoring system
2. Decides whether to handle it directly or decompose into parallel work units
3. Routes each unit to the right model tier (Opus/Sonnet/Haiku) and effort level
4. Spawns subagents that execute in isolated contexts (preserving your main context window)
5. Consolidates results and runs validation on any code output
6. Reports back with a structured summary of what was done

The system is optimized for a Pro plan — it minimizes token waste through context-economy
rules, effort-level tuning, and thinking-token awareness on Opus.

## What's In the Archive

```
orchestra/
├── CLAUDE.md              # The decision engine (orchestration rules, routing, thresholds)
├── INSTALL.md             # This file
├── QUICKREF.md            # One-page cheat sheet for daily use
└── agents/                # Subagent definitions (10 agents)
    ├── orchestrator-planner.md   # Opus, high effort — decomposes tasks
    ├── code-worker.md            # Sonnet, medium effort — generic code worker
    ├── research-worker.md        # Sonnet, medium effort — investigates (read-only)
    ├── project-worker.md         # Haiku, low effort — file ops and scaffolding
    ├── validator.md              # Haiku, low effort — lint/build/test checks
    ├── code-reviewer.md          # Sonnet, medium effort — code review (read-only)
    ├── astro-supabase-worker.md  # Sonnet, medium effort — Astro + Supabase stack
    ├── swiftui-worker.md         # Sonnet, medium effort — SwiftUI / SwiftData / MV
    ├── react-worker.md           # Sonnet, medium effort — React (non-Next.js)
    └── nextjs-worker.md          # Sonnet, medium effort — Next.js App Router
```

## Prerequisites

Before installing, confirm you have:

- **Claude Code** installed and functional in your terminal
- **Pro plan** active (provides access to Opus 4.6, Sonnet 4.6, and Haiku 4.5)
- **Overage credits enabled** (allows Orchestra tasks to complete uninterrupted if you
  hit a usage window reset mid-execution)
- **`--dangerously-skip-permissions` mode** enabled (Orchestra subagents need to read,
  write, and execute without per-action approval prompts)

To verify your Claude Code setup:

```bash
# Check Claude Code is installed
claude --version

# Check your current model (should show opus or sonnet)
# Once in a session, run:
/status
```

---

## Installation Steps

### Step 1: Extract the Archive

If you downloaded the `.tar.gz`:

```bash
# Extract to a temporary location
cd ~/Downloads  # or wherever you saved it
tar -xzf orchestra-system-v3.tar.gz

# You should now have an orchestra/ directory
ls orchestra/
```

If you received the individual files, place them in a folder called `orchestra/` with
the directory structure shown above.

### Step 2: Back Up Your Existing CLAUDE.md

If you already have a global CLAUDE.md, back it up first:

```bash
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.backup 2>/dev/null
echo "Backup complete (or no existing file found — that's fine)"
```

### Step 3: Install the Global CLAUDE.md

You have two options depending on whether you already have a global CLAUDE.md:

**Option A — You have NO existing global CLAUDE.md:**

```bash
cp orchestra/CLAUDE.md ~/.claude/CLAUDE.md
```

**Option B — You ALREADY have a global CLAUDE.md:**

Append the Orchestra rules to the end of your existing file:

```bash
echo "" >> ~/.claude/CLAUDE.md
echo "# ============================================" >> ~/.claude/CLAUDE.md
echo "# ORCHESTRA ORCHESTRATION SYSTEM RULES BELOW" >> ~/.claude/CLAUDE.md
echo "# ============================================" >> ~/.claude/CLAUDE.md
echo "" >> ~/.claude/CLAUDE.md
cat orchestra/CLAUDE.md >> ~/.claude/CLAUDE.md
```

The Orchestra rules are designed to coexist with existing instructions. They add a
decision framework on top of whatever workflow rules you already have. Your existing
CLAUDE.md content (coding conventions, project context, etc.) will still be respected.

### Step 4: Install the Subagents

```bash
# Create the global agents directory if it doesn't exist
mkdir -p ~/.claude/agents

# Copy all Orchestra agents
cp orchestra/agents/*.md ~/.claude/agents/
```

This installs agents globally — they'll be available in every project you open with
Claude Code. The full agent roster:

**Core agents (used in every project):**

| Agent | Model | Effort | Role | Tools |
|-------|-------|--------|------|-------|
| `orchestrator-planner` | Opus | High | Decomposes complex tasks into work units | Read, Grep, Glob |
| `code-worker` | Sonnet | Medium | Generic code — writes and modifies code | Read, Write, Edit, Bash, Glob, Grep |
| `research-worker` | Sonnet | Medium | Gathers and analyzes information | Read, Grep, Glob, WebFetch, WebSearch |
| `project-worker` | Haiku | Low | File operations, scaffolding, organization | Read, Write, Edit, Bash, Glob, Grep |
| `validator` | Haiku | Low | Automated quality checks (lint/build/test) | Read, Bash, Glob, Grep |
| `code-reviewer` | Sonnet | Medium | Independent code review | Read, Grep, Glob |

**Stack-specific agents (auto-selected when project stack is detected):**

| Agent | Model | Effort | Stack | Key Conventions |
|-------|-------|--------|-------|-----------------|
| `astro-supabase-worker` | Sonnet | Medium | Astro + Supabase | File-based routing, content collections, island architecture, RLS, SSR client patterns |
| `swiftui-worker` | Sonnet | Medium | SwiftUI | MV architecture (no MVVM), SwiftData, @Query, @Model, Apple HIG |
| `react-worker` | Sonnet | Medium | React | Functional components, hooks patterns, state management, TanStack Query |
| `nextjs-worker` | Sonnet | Medium | Next.js | App Router, Server Components, Server Actions, ISR, metadata API |

The orchestrator-planner automatically selects stack-specific agents when it detects
the project's tech stack. You can also invoke them directly with `@astro-supabase-worker`,
`@swiftui-worker`, etc.

### Step 5: Add .orchestra/ to Global Gitignore

The `.orchestra/` directory is workspace scaffolding — it should never be committed:

```bash
# Add to global gitignore
echo ".orchestra/" >> ~/.gitignore_global

# Tell git to use the global gitignore (safe to run if already configured)
git config --global core.excludesfile ~/.gitignore_global
```

### Step 6: (Optional) Copy the Quick Reference

Keep the cheat sheet somewhere accessible:

```bash
cp orchestra/QUICKREF.md ~/.claude/ORCHESTRA-QUICKREF.md
```

---

## Verification

Start a **new** Claude Code session (the agents load at session start):

```bash
claude
```

### Check 1: Agents are visible

```
/agents
```

You should see all six Orchestra agents in the list: `orchestrator-planner`,
`code-worker`, `research-worker`, `project-worker`, `validator`, `code-reviewer`.

If any are missing, verify the files exist in `~/.claude/agents/` and that the YAML
frontmatter at the top of each file is valid (no stray characters before the `---`).

### Check 2: Complexity assessment works

Type this prompt:

```
Assess the complexity of: "Add a dark mode toggle to the site header that persists
the user's preference in localStorage"
```

Claude should respond with something like:
> Assessed complexity: 3 → GUIDED mode. Single component change with local storage,
> straightforward implementation.

### Check 3: Orchestra mode triggers

Type this prompt:

```
Plan this but don't execute: "Rebuild the navigation system to support nested dropdowns,
add breadcrumbs to all pages, create a sitemap generator, and add structured data markup
to every page template"
```

Claude should invoke the `orchestrator-planner` and produce a task manifest with
multiple work units, model tier assignments, and dependency ordering.

### Check 4: Stack-specific agents are detected

Navigate to one of your projects (e.g., a project with `astro.config.mjs`) and type:

```
What agents are available for this project?
```

Claude should identify the stack and confirm that the stack-specific agent (e.g.,
`astro-supabase-worker`) will be preferred over the generic `code-worker`.

---

## Configuration Tuning

### Adjusting the Complexity Threshold

If Orchestra mode triggers too often (feels like overkill), raise the threshold in
`~/.claude/CLAUDE.md`. Find this line:

```
- **Score 6+ → ORCHESTRA MODE**
```

Change `6` to `7` or `8`. If you want more orchestration on smaller tasks, lower it to `5`.

### Changing Model Tiers

Edit the `model:` field in any agent's frontmatter file in `~/.claude/agents/`.

Example — temporarily run code-worker on Opus for a complex refactor:

```yaml
model: opus
```

Or set a global override for ALL subagents via environment variable:

```bash
export CLAUDE_CODE_SUBAGENT_MODEL="claude-sonnet-4-6"
```

This overrides every agent's `model:` field. Useful for cost control or testing.

### Changing Effort Levels

Edit the `effort:` field in any agent's frontmatter. Valid values: `low`, `medium`,
`high`, `max`.

The defaults are tuned for Pro plan efficiency:
- `high` for the planner (decomposition quality matters)
- `medium` for builders and reviewers (standard implementation reasoning)
- `low` for validators and project workers (mechanical tasks)

If you find a worker producing poor output, try bumping its effort level up one notch
before switching to a more expensive model tier.

### Adding Project-Specific Agent Overrides

For project-specific workers, create agents in your project's `.claude/agents/` directory.
These override global agents with the same name within that project only.

Example: A SwiftUI-specific code worker for StackGarden:

```bash
mkdir -p ~/projects/stackgarden/.claude/agents
```

Then create `~/projects/stackgarden/.claude/agents/code-worker.md` with SwiftUI-specific
conventions in its system prompt. The global code-worker will still handle all other
projects.

---

## Usage Quick Reference

| You Say | Orchestra Does |
|---------|---------------|
| (any request) | Auto-assesses complexity, picks SINGLE/GUIDED/ORCHESTRA mode |
| "Just do it" or "Quick and dirty" | Forces SINGLE mode — no orchestration |
| "Full orchestra" or "Go deep on this" | Forces ORCHESTRA mode regardless of score |
| "Plan this but don't execute" | Runs planner only, shows manifest, waits for approval |
| "@orchestrator-planner" | Directly invokes the planner agent |
| "@validator" | Directly invokes validation on recent changes |
| "@code-reviewer" | Directly invokes code review on specified files |

---

## Troubleshooting

**Agents don't appear in `/agents` list:**
- Verify files exist: `ls ~/.claude/agents/*.md`
- Check that each file starts with valid YAML frontmatter (the `---` delimiters must be
  the very first line, no blank lines before them)
- Start a new session — agents load at session start

**Orchestra mode never triggers:**
- Check that the CLAUDE.md content is actually present: `cat ~/.claude/CLAUDE.md | grep "Orchestra"`
- The complexity threshold may be too high for your typical tasks — try lowering from 6 to 5
- Try the explicit override: say "Full orchestra" before your request

**Subagent fails or produces garbage:**
- Check which model tier it's running at — Haiku may struggle with tasks that need Sonnet
- Try bumping the effort level before changing the model
- Check that the agent's `tools:` list includes what it needs (e.g., code-worker needs Bash)

**Hit rate limit mid-Orchestra:**
- Your overage credits will kick in automatically
- If you want to avoid overage charges in the future, run large Orchestra tasks early
  in your usage window (shortly after a reset)

**Workers seem to read too many files:**
- The context-economy rules are in each agent's system prompt, but they're guidance, not
  hard enforcement. If a worker is being wasteful, you can strengthen the language in its
  `.md` file or add specific file scope restrictions to the planner's manifest format.

---

## Uninstalling

```bash
# Remove all Orchestra agents
rm ~/.claude/agents/orchestrator-planner.md
rm ~/.claude/agents/code-worker.md
rm ~/.claude/agents/research-worker.md
rm ~/.claude/agents/project-worker.md
rm ~/.claude/agents/validator.md
rm ~/.claude/agents/code-reviewer.md

# Restore your original CLAUDE.md
cp ~/.claude/CLAUDE.md.backup ~/.claude/CLAUDE.md

# Or if you appended, manually edit ~/.claude/CLAUDE.md and remove everything
# below the "ORCHESTRA ORCHESTRATION SYSTEM RULES BELOW" separator

# Remove the quick reference if you copied it
rm ~/.claude/ORCHESTRA-QUICKREF.md

# Remove .orchestra/ from any projects where it was created
find ~/projects -name ".orchestra" -type d -exec rm -rf {} + 2>/dev/null
```
