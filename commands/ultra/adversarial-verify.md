---
name: ultra:adversarial-verify
description: Defender + Attacker + Auditor triple-check verification producing VERIFICATION.md with verdict
argument-hint: "<phase-number>"
allowed-tools:
  - Read
  - Write
  - Glob
  - Grep
  - Bash
  - Task
---
<objective>
Run a triple-check adversarial verification for a phase. Spawns Defender (advocates for work), Attacker (finds gaps), and Auditor (checks compliance) in parallel. Synthesizes into VERIFICATION.md with a PASS/CONDITIONAL_PASS/FAIL verdict.

Context budget: ~15% orchestrator, 100% fresh per verifier agent.
</objective>

<execution_context>
@get-shit-done/workflows/adversarial-verify.md
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
Execute the adversarial-verify workflow from @get-shit-done/workflows/adversarial-verify.md end-to-end.

1. Initialize and load phase artifacts
2. Spawn 3 verifier agents in parallel (Defender, Attacker, Auditor)
3. Verify all 3 reports written (DEFENSE.md, ATTACK.md, AUDIT.md)
4. Synthesize verdict from all 3 perspectives
5. Write VERIFICATION.md with structured gaps (if any)
6. Commit and present results with next steps

Preserve all workflow gates (parallel spawning, verdict synthesis, gap structuring).
</process>
