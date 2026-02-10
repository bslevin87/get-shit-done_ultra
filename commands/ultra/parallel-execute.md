---
name: ultra:parallel-execute
description: Domain-owned parallel execution with file ownership enforcement
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
Execute all plans in a phase using domain-owned parallel execution. Each executor agent is told which files it owns and must not touch files outside its domain. Uses wave-based execution from GSD with Ultra's file ownership enforcement.

Context budget: ~15% orchestrator, 100% fresh per executor agent.
</objective>

<execution_context>
@get-shit-done/workflows/execute-phase.md
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
Execute using the standard GSD execute-phase workflow with Ultra enhancements:

1. Initialize with gsd-tools.js (same as GSD execute-phase)
2. Discover and group plans by wave (same as GSD)
3. For each plan, enhance the executor prompt with file ownership:

   ```
   FILE OWNERSHIP (from DOMAINS.md + gsd-ultra.json):
   - YOU OWN: {domain_path} (full read/write)
   - SHARED (append-only): {shared_paths}
   - DO NOT TOUCH: {other_domain_paths}

   CRITICAL: If you need to modify a file outside your domain,
   STOP and report it as a deviation. Do NOT modify it.
   ```

4. Spawn executor agents with enhanced prompts (same wave execution)
5. Handle checkpoints (same as GSD)
6. Aggregate results (same as GSD)
7. Skip verification (done separately via /ultra:adversarial-verify)

The key difference from standard GSD execute-phase: file ownership is injected into each executor's prompt, preventing cross-domain file conflicts during parallel execution.
</process>
