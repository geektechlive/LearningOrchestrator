# Contributing to LearningOrchestrator

Thanks for your interest. Contributions are welcome — bug reports, new agent definitions, improved routing logic, or stack-specific workers for additional frameworks.

## How to contribute

This project follows the standard fork & pull request model.

1. **Fork** the repository to your GitHub account
2. **Clone** your fork locally
3. **Create a branch** for your change (`git checkout -b my-change`)
4. **Make your changes** and test them locally (see [Testing](#testing) below)
5. **Push** to your fork and **open a Pull Request** against `main`

All PRs are reviewed before merging. Keep changes focused — one thing per PR.

## What's worth contributing

- **New stack-specific agents** — if you've built a worker for a stack not covered (e.g., Django, Laravel, Flutter), it belongs in `orchestra/agents/`
- **Routing improvements** — better complexity signals, thresholds, or tier assignments
- **Bug reports** — if an agent consistently produces wrong output or the orchestrator routes incorrectly, open an issue with a reproducible example
- **Documentation fixes** — typos, outdated steps, unclear instructions

## What to avoid

- Changes to `CLAUDE.md` that introduce stack-specific conventions — those belong in individual agent definitions
- Adding dependencies or install scripts — this system is intentionally zero-dependency
- Large refactors without an issue discussion first

## Agent file format

Each agent is a Markdown file with YAML frontmatter:

```yaml
---
name: my-worker
description: One-line description used for agent selection
model: sonnet
effort: medium
tools: Read, Write, Edit, Bash, Glob, Grep
---

System prompt here...
```

Place new agents in `orchestra/agents/`. Follow the naming pattern: `<stack>-worker.md` for stack-specific workers, descriptive names for general-purpose agents.

## Testing

Before opening a PR:

1. Install the agents locally (`cp orchestra/agents/*.md ~/.claude/agents/`)
2. Start a new Claude Code session (`claude`)
3. Verify your agent appears in `/agents`
4. Run a task that exercises the agent and confirm the output is correct

There are no automated tests — manual verification is the current standard.

## Opening issues

Use GitHub Issues for bugs and feature requests. Include:
- What you expected to happen
- What actually happened
- The prompt or task that triggered it
- Your Claude Code version (`claude --version`)
