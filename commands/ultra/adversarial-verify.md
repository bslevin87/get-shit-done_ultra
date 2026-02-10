---
name: ultra:adversarial-verify
description: 3-round debate verification — Defender + Attacker + Auditor produce VERIFICATION.md with verdict
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
Run a 3-round adversarial verification for a phase.

**Round 1: Independent Assessment** (parallel) — Defender, Attacker, and Auditor each assess independently.
**Round 2: Structured Debate** — Attacker presents findings → Defender responds (Concede/Dispute/Mitigate) → Auditor rules.
**Round 3: Consensus** (if needed) — Only for unresolved disputes.

Synthesizes into VERIFICATION.md with a PASS/CONDITIONAL_PASS/FAIL verdict.

Context budget: ~20% orchestrator, 100% fresh per verifier agent per round.
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
2. **Round 1:** Spawn 3 verifier agents in parallel (Defender→DEFENSE.md, Attacker→ATTACK.md, Auditor→AUDIT.md)
3. **Round 2:** Structured debate — Attacker presents ATK-{N} findings, Defender responds with Concede/Dispute/Mitigate, Auditor rules on disputes
4. **Round 3:** (if needed) Consensus round for unresolved disputes only
5. **Consensus Detection:** Issues flagged by all 3 = very strong signal
6. Synthesize enhanced VERIFICATION.md with executive summary, evidence matrix, debate results, compliance score, verdict
7. Commit and present results with next steps

Preserve all workflow gates (parallel spawning, debate protocol, verdict synthesis, gap structuring).
</process>
