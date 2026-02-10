---
name: ultra:adversarial-plan
description: Builder/Critic adversarial debate loop until APPROVED or max 5 rounds
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
Run a Builder/Critic adversarial planning debate for a phase. Builder creates plans with domain decomposition and file ownership. Critic reviews for gaps, risks, and quality issues. Loop continues until zero BLOCKERs (APPROVED) or max 5 rounds.

Context budget: ~15% orchestrator, 100% fresh per Builder/Critic agent.
</objective>

<execution_context>
@get-shit-done/workflows/adversarial-plan.md
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
Execute the adversarial-plan workflow from @get-shit-done/workflows/adversarial-plan.md end-to-end.

1. Initialize and load research (from research-swarm)
2. Spawn Builder to create initial plans
3. Spawn Critic to review plans
4. If BLOCKERs: send feedback to Builder, repeat (max 5 rounds)
5. If APPROVED: commit plans and present results
6. If max rounds: present remaining issues, ask user

Preserve all workflow gates (debate loop, convergence detection, max rounds).
</process>
