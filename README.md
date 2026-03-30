# Claude Code Review Agents

A two-round, multi-agent code review pipeline for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Six specialist agents review your branch diff in parallel, then cross-examine each other's findings to surface consensus, contradictions, and blind spots.

## What's in the box

### Pipeline commands (`.claude/commands/`)

| Command | Description |
|---------|-------------|
| `usuals-v2` | Full pipeline: context gathering → 6-agent review → cross-examination → consolidated report → post-fix verification |
| `usuals` | Same pipeline using v1 agent definitions (more verbose prompts) |

### Agents (`.claude/`)

**Pipeline orchestration:**

| Agent | Role |
|-------|------|
| `branch-context-analyst-v2` | Pre-review: extracts intent, decisions, and trade-offs from conversation history + git data |
| `post-fix-verifier-v2` | Post-fix: runs tests, validates branch intent survived the fixes, flags coverage gaps |

**Review panel (6 agents, v1 + v2 variants):**

| Agent | Focus |
|-------|-------|
| `code-reviewer` | Correctness, readability, duplication, best practices |
| `laravel-specialist` | Eloquent patterns, queue/job design, Blade/Livewire, Laravel conventions |
| `performance-engineer` | Query performance, caching, memory, scalability |
| `code-quality-pragmatist` | Over-engineering, unnecessary abstraction, YAGNI violations |
| `penetration-tester` | OWASP Top 10, auth gaps, injection, data exposure |
| `chaos-engineer` | Failure modes, error handling gaps, race conditions, resilience |

**Utility agents:**

| Agent | Role |
|-------|------|
| `branch-scope-test-runner` | Maps changed files to tests and runs them |
| `php-pro` | PHP 8.x language-level review |

**Personality agents (fun, optional):**

| Agent | Vibe |
|-------|------|
| `Jenny` | Specification compliance auditor |
| `karen` | Project reality manager / status assessment |
| `vex` | Cyberpunk-flavored reviewer |

## How it works

The pipeline runs in 7 steps:

1. **Context gathering** — `branch-context-analyst-v2` reads conversation history, branch name, and commit log to produce a Branch Context Brief
2. **Diff collection** — `git diff main...HEAD`
3. **Round 1** — All 6 review agents launch in parallel, each receiving the context brief + diff
4. **Round 2** — All 6 agents relaunch with everyone's Round 1 findings, reacting from their own lens (agree, disagree, escalate, dismiss)
5. **Consolidation** — Findings grouped into: Consensus, Escalated, Single-Agent, Contradictions, Dismissed
6. **Report** — Presented to user, who chooses what to fix
7. **Verification** — After fixes, `post-fix-verifier-v2` runs tests and validates the branch still achieves its original intent

## Installation

Copy the `.claude/` directory into your project root:

```bash
cp -r .claude/ /path/to/your/project/.claude/
```

Make sure `.claude/` is **not** in your `.gitignore` if you want these tracked in your repo.

## Usage

In Claude Code, run:

```
/usuals-v2
```

Or say "run the usuals" in conversation after doing some work on a branch.

## Customization

- **v2 agents** are concise and generic — designed for any Laravel project
- **v1 agents** are more verbose with detailed methodology sections
- Add project-specific notes to any agent by editing the relevant `.md` file
- The `post-fix-verifier-v2` safety rules should be reviewed for your project's test DB naming convention

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- A git repository with a `main` branch
- For Laravel-specific agents: a Laravel project with Pest or PHPUnit

## License

MIT
