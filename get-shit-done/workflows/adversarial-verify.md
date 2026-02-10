<purpose>
Execute a 3-round adversarial verification with Defender, Attacker, and Auditor agents.

**Round 1:** Independent assessment (parallel) — each agent evaluates the phase independently.
**Round 2:** Structured debate — Attacker presents findings, Defender responds, Auditor rules.
**Round 3:** Consensus (if needed) — only for unresolved disputes after Round 2.

The 3-round structure produces better verdicts than parallel-only verification because agents respond to each other's evidence rather than working in isolation.
</purpose>

<core_principle>
Three rounds catch what one round misses:

- **Round 1** catches the obvious: stubs, missing files, convention violations
- **Round 2** catches the subtle: whether "evidence" actually proves the claim, whether "findings" are real issues or misunderstandings
- **Round 3** catches the contentious: issues where reasonable agents disagree, requiring structured consensus

Consensus detection: when all 3 roles flag the same issue independently in Round 1, it's a very strong signal — the most reliable finding in the entire process.
</core_principle>

<process>

<step name="initialize" priority="first">
Load all context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init verify-work "${PHASE_ARG}" --include state,roadmap)
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `verifier_model`.

Resolve models for verification roles:
```bash
ULTRA_CONFIG=$(cat gsd-ultra.json 2>/dev/null || echo '{}')
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_profile || 'balanced'")

DEFENDER_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.defender['$MODEL_PROFILE']")
ATTACKER_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.attacker['$MODEL_PROFILE']")
AUDITOR_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.auditor['$MODEL_PROFILE']")
```

Fallback: use `gsd-verifier` model for all 3.
</step>

<step name="detect_agent_teams" priority="after-init">
Check if Agent Teams is available for native multi-agent coordination:

1. Check env: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`
   ```bash
   AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
   ```
2. Verify Claude Code supports Agent Teams (Opus 4.6+ with experimental flag)

Set coordination mode:
- `AGENT_TEAMS_MODE=true` → live debate via native teammate messaging (biggest win)
- `AGENT_TEAMS_MODE=false` → Task() subagents + debate moderator (existing behavior, default)

```bash
if [ "$AGENT_TEAMS_ENV" = "1" ]; then
  AGENT_TEAMS_MODE=true
else
  AGENT_TEAMS_MODE=false
fi
```

Display:
```
◆ Coordination: Agent Teams (native)    ← if AGENT_TEAMS_MODE=true
◆ Coordination: Task() subagents        ← if AGENT_TEAMS_MODE=false
```

**Both paths produce identical outputs** — same DEFENSE/ATTACK/AUDIT/DEBATE.md files, same VERIFICATION.md, same verdict logic. Agent Teams mode replaces file-serialized debate with live messaging — the biggest win in this upgrade.
</step>

<step name="prepare">
Read phase artifacts:
```bash
ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null
ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null
```

Build verification context:
```markdown
<verification_context>
**Phase:** {phase_number} - {phase_name}
**Goal:** {phase goal from ROADMAP.md}
**Phase directory:** {phase_dir}

Read the PLAN.md, SUMMARY.md files in the phase directory.
Read CLAUDE.md, DOMAINS.md, REQUIREMENTS.md from project root.
Read gsd-ultra.json for pipeline configuration.
Examine the actual source code in src/.
</verification_context>
```
</step>

<step name="round_1_independent">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► ADVERSARIAL VERIFICATION — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Round 1: Independent Assessment (parallel)
  → Defender  ({DEFENDER_MODEL}) — building the case FOR
  → Attacker  ({ATTACKER_MODEL}) — finding what's BROKEN
  → Auditor   ({AUDITOR_MODEL}) — checking COMPLIANCE
```

Spawn all 3 in parallel:

```
Task(
  prompt="First, read the ultra-defender agent definition.
  Then build a defense for Phase {phase}: {phase_name}.
  {verification_context}
  Write DEFENSE.md to: {phase_dir}/
  Use DEF-{N} IDs for all evidence points.
  Include an evidence matrix with file:line references.",
  subagent_type="general-purpose",
  model="{DEFENDER_MODEL}",
  description="Defend Phase {phase}"
)

Task(
  prompt="First, read the ultra-attacker agent definition.
  Then attack Phase {phase}: {phase_name}.
  {verification_context}
  Write ATTACK.md to: {phase_dir}/
  Use ATK-{N} IDs for all findings.
  Tag each finding with composite perspective (critic/security-auditor/edge-case-hunter).",
  subagent_type="general-purpose",
  model="{ATTACKER_MODEL}",
  description="Attack Phase {phase}"
)

Task(
  prompt="First, read the ultra-auditor agent definition.
  Then audit Phase {phase}: {phase_name}.
  {verification_context}
  Write AUDIT.md to: {phase_dir}/
  Calculate compliance score with weighted dimensions.",
  subagent_type="general-purpose",
  model="{AUDITOR_MODEL}",
  description="Audit Phase {phase}"
)
```

Wait for all 3 to complete.

Verify outputs:
```bash
for f in DEFENSE ATTACK AUDIT; do
  [ -f "${PHASE_DIR}/${f}.md" ] && echo "✓ ${f}.md" || echo "✗ ${f}.md MISSING"
done
```
</step>

<step name="consensus_detection">
Before debate, check for consensus findings — issues flagged by all 3 roles:

Read DEFENSE.md (honest gaps), ATTACK.md (findings), AUDIT.md (non-compliance).
Cross-reference to find issues all 3 mention.

```markdown
## Pre-Debate Consensus
{N} issues flagged by all 3 roles → VERY STRONG signal

| Issue | Defender | Attacker | Auditor |
|-------|----------|----------|---------|
| {issue} | Honest Gap | ATK-{N} | Non-compliance |
```

Consensus findings skip debate — they're automatically confirmed.
</step>

<step name="round_2_debate">
Display:
```
◆ Round 2: Structured Debate
  → Attacker presents {N} findings
  → Defender responds to each
  → Auditor rules on disputes
```

**Only run debate if Attacker has HIGH or CRITICAL findings that aren't consensus findings.**

If all Attacker findings are LOW/MEDIUM, or all are consensus findings, skip to verdict.

Run debate as a single agent call with all 3 reports as context:

```
Task(
  prompt="You are the debate moderator for Phase {phase} adversarial verification.

  Read these 3 reports:
  - DEFENSE.md (DEF-{N} evidence)
  - ATTACK.md (ATK-{N} findings)
  - AUDIT.md (compliance score)

  Run the structured debate:

  For each ATK-{N} finding (HIGH or CRITICAL, non-consensus):
  1. Attacker presents: severity, evidence, challenge to Defender
  2. Defender responds: CONCEDE or DISPUTE (with file:line counter-evidence)
  3. Auditor rules: SUSTAINED (Attacker wins) / OVERRULED (Defender wins) / SPLIT

  Write DEBATE.md to: {phase_dir}/

  Format:
  # Debate Results — Phase {X}

  ## Debated Findings
  | ATK-ID | Attacker | Defender Response | Auditor Ruling | Final Severity |
  |--------|----------|-------------------|----------------|----------------|

  ## Detailed Debate
  ### ATK-{N}: {title}
  **Attacker:** {evidence}
  **Defender:** {CONCEDE|DISPUTE} — {reasoning}
  **Auditor:** {SUSTAINED|OVERRULED|SPLIT} — {basis}
  **Final severity:** {adjusted}

  ## Consensus Findings (auto-confirmed)
  {list from pre-debate consensus}

  ## Debate Summary
  - Findings sustained: {N}
  - Findings overruled: {N}
  - Findings split: {N}
  - Consensus findings: {N}",
  subagent_type="general-purpose",
  model="{ATTACKER_MODEL}",
  description="Debate Phase {phase}"
)
```
</step>

<step name="round_3_consensus">
**Only if Round 2 has unresolved disputes** (SPLIT rulings where severity is contested).

Display:
```
◆ Round 3: Consensus (resolving {N} disputes)
```

For each SPLIT ruling, the orchestrator makes a final call based on:
1. Weight of evidence (file:line references vs. assertions)
2. Severity precedent (safety concerns > style concerns)
3. Consensus overlap (if 2 of 3 agree, that wins)

Most phases won't need Round 3. It's the safety valve.
</step>

<step name="synthesize_verdict">
Display:
```
◆ Synthesizing verdict from 3 perspectives + debate...
```

Read all reports (DEFENSE.md, ATTACK.md, AUDIT.md, DEBATE.md if exists).

**Verdict logic (same thresholds, enhanced with debate adjustments):**

After debate, some findings are withdrawn (overruled) or adjusted (split). Use adjusted findings:

**PASS** — All of:
- Defender: ≥90% must-haves verified with STRONG evidence
- Attacker: 0 critical findings (post-debate), ≤2 high findings (post-debate)
- Auditor: ≥90% compliance score
- Consensus: 0 consensus findings unresolved

**CONDITIONAL_PASS** — All of:
- Defender: ≥70% must-haves verified
- Attacker: 0 critical findings (post-debate) (highs OK)
- Auditor: ≥70% compliance score
- No single issue flagged by all 3 roles remains unresolved

**FAIL** — Any of:
- Defender: <70% must-haves verified
- Attacker: ≥1 critical finding (post-debate)
- Auditor: <70% compliance
- Consensus failure remains unresolved

Write enhanced VERIFICATION.md:

```markdown
---
phase: {phase_dir_name}
verified: {timestamp}
status: {passed|gaps_found|human_needed}
verdict: {PASS|CONDITIONAL_PASS|FAIL}
score: {N}/{M} must-haves verified
defense_score: {defender_pct}%
attack_findings: {critical}/{high}/{medium}/{low}
attack_findings_post_debate: {critical}/{high}/{medium}/{low}
audit_compliance: {auditor_pct}%
debate_rounds: {1|2|3}
consensus_findings: {count}
gaps:
  - truth: "{failed truth}"
    status: failed
    reason: "{evidence from attacker}"
    artifacts:
      - path: "{file}"
        issue: "{from attacker or auditor}"
    missing:
      - "{what needs fixing}"
---

# Phase {X}: {Name} — Adversarial Verification Report

**Phase Goal:** {goal}
**Verified:** {timestamp}
**Verdict:** {PASS | CONDITIONAL_PASS | FAIL}

## Executive Summary

{2-3 sentence summary of the verification outcome, highlighting key strengths and weaknesses}

## Verdict Rationale

### Defender Assessment
{summary from DEFENSE.md}
Score: {N}/{M} must-haves ({pct}%)
Evidence strength: STRONG: {N}, MODERATE: {M}, WEAK: {P}

### Attacker Assessment
{summary from ATTACK.md}
Findings (pre-debate): {critical} critical, {high} high, {medium} medium, {low} low
Findings (post-debate): {critical} critical, {high} high, {medium} medium, {low} low

### Auditor Assessment
{summary from AUDIT.md}
Compliance: {pct}%

## Evidence Matrix (from Defender)

| ID | Requirement | Evidence | File:Line | Status |
|----|-------------|----------|-----------|--------|
{from DEFENSE.md evidence matrix}

## Debate Results (Round 2)

| ATK-ID | Finding | Defender Response | Auditor Ruling | Final Severity |
|--------|---------|-------------------|----------------|----------------|
{from DEBATE.md}

## Consensus Findings

{issues flagged by all 3 — strongest signals}

## Must-Have Status (Combined)

| Must-Have | Defender | Attacker | Auditor | Debate | Final |
|-----------|----------|----------|---------|--------|-------|
| {truth} | {status} | {status} | {status} | {ruling} | {status} |

## Gaps (If Any)
{structured gaps in YAML frontmatter format for /ultra:gap-close}

## Human Verification Needed
{items requiring human testing}

## Recommendations
{what to do next based on verdict}
```

Write to: `${PHASE_DIR}/${PADDED_PHASE}-VERIFICATION.md`
</step>

<step name="commit">
```bash
FILES="${PHASE_DIR}/DEFENSE.md ${PHASE_DIR}/ATTACK.md ${PHASE_DIR}/AUDIT.md ${PHASE_DIR}/${PADDED_PHASE}-VERIFICATION.md"
[ -f "${PHASE_DIR}/DEBATE.md" ] && FILES="$FILES ${PHASE_DIR}/DEBATE.md"

node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(ultra): adversarial verify phase ${PHASE} — ${VERDICT}" --files $FILES
```
</step>

</process>

<offer_next>
Output based on verdict:

**If PASS:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► VERIFICATION PASSED ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phase {X}: {Name} — PASS

| Role | Score | Summary |
|------|-------|---------|
| Defender | {N}/{M} ({pct}%) | {1-line} |
| Attacker | {findings post-debate} | {1-line} |
| Auditor | {pct}% | {1-line} |

Debate: {N} findings debated, {M} sustained, {P} overruled
Consensus: {N} consensus findings (all resolved)

Phase complete! Run the knowledge flywheel:

/ultra:retrospective {X}
```

**If CONDITIONAL_PASS:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► CONDITIONAL PASS ⚠
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phase {X}: {Name} — CONDITIONAL_PASS

Conditions:
{list of conditions/warnings}

Options:
1. Accept and proceed → /ultra:retrospective {X}
2. /ultra:gap-close {X} — fix remaining issues
```

**If FAIL:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► VERIFICATION FAILED ✗
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phase {X}: {Name} — FAIL

Critical Issues:
{from attacker's critical findings post-debate}

Consensus Failures:
{from consensus findings}

Gaps:
{from synthesized gaps}

───────────────────────────────────────────────────────

## ▶ Next Up

**Gap Close** — fix gaps and re-verify

/ultra:gap-close {X}

<sub>/clear first → fresh context window</sub>
```
</offer_next>

<success_criteria>
- [ ] Round 1: All 3 verifier agents spawned in parallel
- [ ] Round 1: DEFENSE.md, ATTACK.md, AUDIT.md written with IDs
- [ ] Consensus detection: cross-role issues identified
- [ ] Round 2: Structured debate for HIGH/CRITICAL findings (if applicable)
- [ ] Round 2: DEBATE.md written with rulings
- [ ] Round 3: Unresolved disputes resolved (if needed)
- [ ] Verdict synthesized from all perspectives + debate results
- [ ] Enhanced VERIFICATION.md written with executive summary, evidence matrix, debate results
- [ ] All reports committed to git
- [ ] User knows verdict and next steps
</success_criteria>
