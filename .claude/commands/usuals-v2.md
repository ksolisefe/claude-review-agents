# The Usuals v2 — Two-Round Review

Run the standard 6-agent review pipeline against the latest changes, then feed all findings back to each agent so they can react to the full picture.

## Instructions

**Step 1 — Gather branch context**

Launch the **branch-context-analyst-v2** agent. This agent:
- Reads the branch name, commit log, and file change summary
- Reviews the full conversation history (what the user asked for, problems encountered, approaches tried and abandoned, trade-offs made, known concerns)
- Produces a structured **Branch Context Brief**

If the conversation is short or thin (e.g., user just said "run the usuals" with no prior discussion), the analyst will flag what it couldn't infer — ask the user to fill in the gaps before proceeding.

Wait for the Branch Context Brief before moving to Step 2.

**Step 2 — Get the diff**

Run `git diff main...HEAD` (full branch) or `git diff HEAD~1 HEAD` (last commit) to get the actual code changes.

**Step 3 — Round 1: Launch all 6 agents in parallel**

Launch these 6 agents simultaneously. Each agent receives:
- The **Branch Context Brief** from Step 1 (prepended at the top of their prompt)
- The **diff** from Step 2

Agents:

1. **code-reviewer-v2** — Correctness, readability, duplication, general best practices
2. **laravel-specialist-v2** — Eloquent patterns, queue/job design, Blade/Livewire, Laravel conventions
3. **performance-engineer-v2** — Query performance, caching, memory, scalability
4. **code-quality-pragmatist-v2** — Over-engineering, unnecessary abstraction, YAGNI violations
5. **penetration-tester-v2** — OWASP Top 10, auth gaps, injection, data exposure
6. **chaos-engineer-v2** — Failure modes, error handling gaps, race conditions, resilience

Each agent should return findings as: **[SEVERITY: HIGH/MED/LOW]** — description — file:line

**Step 4 — Round 2: Cross-feed all findings back to all 6 agents in parallel**

Collect all Round 1 reports into a single combined summary. Then re-launch all 6 agents simultaneously, giving each agent:
- The **Branch Context Brief** from Step 1
- The **original diff** (same as Round 1)
- The **complete Round 1 findings** from all 6 agents

Each agent should respond from their own perspective:
- **Agree or disagree** with findings raised by the other agents — explain why
- **Escalate or downgrade** any finding where their domain expertise gives them a stronger view
- **Flag anything missed** that the other agents' findings make more visible in hindsight
- **Call out contradictions** between agents and state which view is correct from their lens

Each agent returns a short Round 2 report: only new information, escalations, disagreements, or corrections — not a repeat of their Round 1 findings.

**Step 5 — Produce the final consolidated list**

After all Round 2 reports are back, compile the final output:

```
## CONSENSUS (flagged or confirmed by 2+ agents)
[HIGH] ...

## ESCALATED IN ROUND 2
[HIGH] (was MED/LOW in Round 1) ...

## SINGLE-AGENT (credible, no contradiction)
[MED] ...
[LOW] ...

## CONTRADICTIONS
- Agent A said X, Agent B disagreed — verdict: ...

## DISMISSED
- Finding X: majority of agents pushed back — reason: ...
```

**Step 6 — Report back**
Output the consolidated findings. Then ask: "Want me to implement all of these, or just the HIGH items?"

**Step 7 — Post-fix verification (after fixes are applied)**

Once the user approves and review fixes have been implemented, launch the **post-fix-verifier-v2** agent. This agent receives:
- The **Branch Context Brief** from Step 1
- A summary of the **review fixes that were just applied**

The verifier will:
1. Run a safety pre-flight check (confirm test DB config is correct)
2. Run branch-scoped tests (tests related to changed files)
3. Run the full test suite as a regression smoke test
4. Validate that the branch still achieves its original intent (the most important check — review fixes sometimes accidentally undo the very change the branch was built for)
5. Flag coverage gaps for changed files without tests

If the verifier returns **FAIL**, do not commit. Report the failures and ask the user how to proceed.
If it returns **PASS** or **PASS WITH WARNINGS**, present the report and ask: "Ready to commit?"
