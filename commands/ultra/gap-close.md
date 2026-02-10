---
name: ultra:gap-close
description: Bug clustering + ralph self-verify — reads VERIFICATION.md gaps, clusters fixes, spawns fixer agents with self-verification
argument-hint: "<phase-number>"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
  - Task
  - AskUserQuestion
---
<objective>
Close gaps found by adversarial verification. Reads VERIFICATION.md gaps, clusters bugs intelligently, selects appropriate agent types, spawns fixers with self-verification (Ralph protocol), and retries up to 5 times if gaps persist.

Context budget: ~20% orchestrator, 100% fresh per fixer agent.
</objective>

<execution_context>
@get-shit-done/workflows/gap-close.md
</execution_context>

<context>
Phase: $ARGUMENTS

@.planning/ROADMAP.md
@.planning/STATE.md
@CLAUDE.md
@DOMAINS.md
@gsd-ultra.json
</context>

<process>
Execute the gap-close workflow from @get-shit-done/workflows/gap-close.md end-to-end.

1. Load gaps from VERIFICATION.md (including debate results if present)
2. **Bug Clustering** — group gaps by file/domain, assign to clusters
3. **Agent Type Selection** — build errors→executor, logic bugs→debugger, missing features→executor, CSS→UI-executor
4. **Spawn Fixers** — one per cluster, with file ownership and Ralph self-verify protocol
5. **Ralph 3-Level Self-Verify** — each fixer verifies its own work (code review, logical walk-through, runtime check)
6. **Lead Integration Check** — conflict check, build verify, test suite, spot check
7. **Retry Logic** — up to 5 attempts, with escalation analysis on exhaustion
8. Update VERIFICATION.md with results
</process>
