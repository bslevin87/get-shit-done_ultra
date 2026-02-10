<purpose>
Execute a triple-check adversarial verification with Defender, Attacker, and Auditor agents. Each provides a different perspective on whether the phase goal was achieved. Their reports are synthesized into a VERIFICATION.md with a verdict.

Replaces GSD's single verifier with a 3-role adversarial system: the Defender advocates for the work, the Attacker tries to break it, and the Auditor performs neutral compliance checking.
</purpose>

<core_principle>
Three perspectives prevent blind spots. A single verifier might miss stubs (optimistic) or flag non-issues (pessimistic). With Defender + Attacker + Auditor:
- Defender ensures we recognize what works (prevents unnecessary rework)
- Attacker ensures we find what's broken (prevents false passes)
- Auditor ensures we check compliance (prevents drift from standards)

The synthesis weighs all three to produce a fair verdict.
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

<step name="spawn_verifiers">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► ADVERSARIAL VERIFICATION — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning 3 verifiers in parallel...
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
  Write DEFENSE.md to: {phase_dir}/",
  subagent_type="general-purpose",
  model="{DEFENDER_MODEL}",
  description="Defend Phase {phase}"
)

Task(
  prompt="First, read the ultra-attacker agent definition.
  Then attack Phase {phase}: {phase_name}.
  {verification_context}
  Write ATTACK.md to: {phase_dir}/",
  subagent_type="general-purpose",
  model="{ATTACKER_MODEL}",
  description="Attack Phase {phase}"
)

Task(
  prompt="First, read the ultra-auditor agent definition.
  Then audit Phase {phase}: {phase_name}.
  {verification_context}
  Write AUDIT.md to: {phase_dir}/",
  subagent_type="general-purpose",
  model="{AUDITOR_MODEL}",
  description="Audit Phase {phase}"
)
```

Wait for all 3 to complete.
</step>

<step name="verify_outputs">
```bash
for f in DEFENSE ATTACK AUDIT; do
  [ -f "${PHASE_DIR}/${f}.md" ] && echo "✓ ${f}.md" || echo "✗ ${f}.md MISSING"
done
```

If any missing: report which verifier failed, use available reports.
</step>

<step name="synthesize_verdict">
Display:
```
◆ Synthesizing verdict from 3 perspectives...
```

Read all 3 reports. Weigh evidence:

**Verdict logic:**
1. Parse Defender's must-have scores
2. Parse Attacker's critical/high findings
3. Parse Auditor's compliance score

**PASS** — All of:
- Defender: ≥90% must-haves verified with STRONG evidence
- Attacker: 0 critical findings, ≤2 high findings
- Auditor: ≥90% compliance score

**CONDITIONAL_PASS** — All of:
- Defender: ≥70% must-haves verified
- Attacker: 0 critical findings (highs OK)
- Auditor: ≥70% compliance score
- AND no single issue appears in all 3 reports

**FAIL** — Any of:
- Defender: <70% must-haves verified
- Attacker: ≥1 critical finding
- Auditor: <70% compliance
- OR same issue flagged by all 3 (consensus failure)

Write VERIFICATION.md:

```markdown
---
phase: {phase_dir_name}
verified: {timestamp}
status: {passed|gaps_found|human_needed}
verdict: {PASS|CONDITIONAL_PASS|FAIL}
score: {N}/{M} must-haves verified
defense_score: {defender_pct}%
attack_findings: {critical}/{high}/{medium}/{low}
audit_compliance: {auditor_pct}%
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

## Verdict Rationale

### Defender Assessment
{summary from DEFENSE.md}
Score: {N}/{M} must-haves ({pct}%)

### Attacker Assessment
{summary from ATTACK.md}
Findings: {critical} critical, {high} high, {medium} medium, {low} low

### Auditor Assessment
{summary from AUDIT.md}
Compliance: {pct}%

## Consensus Analysis
{areas where all 3 agree — strongest signals}
{areas of disagreement — explain discrepancies}

## Must-Have Status (Combined)

| Must-Have | Defender | Attacker | Auditor | Final |
|-----------|----------|----------|---------|-------|
| {truth} | {status} | {status} | {status} | {status} |

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
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(ultra): adversarial verify phase ${PHASE} — ${VERDICT}" --files "${PHASE_DIR}/DEFENSE.md" "${PHASE_DIR}/ATTACK.md" "${PHASE_DIR}/AUDIT.md" "${PHASE_DIR}/${PADDED_PHASE}-VERIFICATION.md"
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
| Attacker | {findings} | {1-line} |
| Auditor | {pct}% | {1-line} |

Phase complete! Proceed to next phase.
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
1. Accept and proceed
2. /ultra:gap-close {X} — fix remaining issues
```

**If FAIL:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► VERIFICATION FAILED ✗
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phase {X}: {Name} — FAIL

Critical Issues:
{from attacker's critical findings}

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
- [ ] All 3 verifier agents spawned in parallel
- [ ] DEFENSE.md, ATTACK.md, AUDIT.md written
- [ ] Verdict synthesized from all 3 perspectives
- [ ] VERIFICATION.md written with structured gaps (if any)
- [ ] Consensus and disagreements documented
- [ ] All reports committed to git
- [ ] User knows verdict and next steps
</success_criteria>
