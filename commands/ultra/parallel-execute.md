---
name: ultra:parallel-execute
description: Domain-owned parallel execution with deferred integration and two-layer parallelism
argument-hint: "<phase-number> [--gaps-only]"
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
Execute all plans in a phase using domain-owned parallel execution with deferred integration. Each executor agent owns specific files and must not touch files outside its domain. Uses two-layer parallelism: teammates work domains in parallel (Layer 1), and teammates can spawn subagents within their domain (Layer 2).

Context budget: ~15% orchestrator (lead), 100% fresh per executor agent (teammate).
</objective>

<execution_context>
@get-shit-done/workflows/parallel-execute.md
</execution_context>

<context>
Phase: $ARGUMENTS

**Flags:**
- `--gaps-only` — Execute only gap closure plans (plans with `gap_closure: true`)

@.planning/ROADMAP.md
@.planning/STATE.md
@CLAUDE.md
@DOMAINS.md
@gsd-ultra.json
</context>

<process>
Execute using the parallel-execute workflow with Ultra's deferred integration model:

1. **Initialize** — Load phase, discover plans, group by wave
2. **Assign Ownership** — Each plan gets explicit file ownership (OWN/SHARED/READ-ONLY/DO NOT TOUCH)
3. **Layer 1: Domain Execution** — Spawn teammate agents per domain, all in parallel within a wave
4. **Layer 2: Sub-Execution** — Teammates may spawn subagents for tasks within their owned domain
5. **Deferred Integration** — After all teammates complete, lead integrates serially (wiring, shared state, cross-domain connections)
6. **Hub-Spoke Communication** — All cross-domain discoveries go through lead, not peer-to-peer
7. **Aggregate Results** — Collect summaries, handle checkpoints

The key difference from standard GSD execute-phase: deferred integration prevents cross-domain conflicts, two-layer parallelism maximizes throughput, and hub-spoke communication prevents coordination chaos.

Sweet spot: 4-5 teammates. Interface contracts required for 3+ domains.
</process>
