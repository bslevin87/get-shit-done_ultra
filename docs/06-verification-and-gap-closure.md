# Verification and Gap Closure

Verification and gap closure are one flow: find problems, then fix them. Ultra's triple-check debate catches what single-verifier systems miss, and bug clustering fixes root causes instead of symptoms.

## Adversarial Verification

### Round 1: Independent Assessment (Parallel)

Three agents assess the implementation independently, without seeing each other's work:

| Agent | Output | Focus |
|-------|--------|-------|
| **Defender** | DEFENSE.md | Evidence that implementation meets requirements |
| **Attacker** | ATTACK.md | Bugs, vulnerabilities, missing functionality |
| **Auditor** | AUDIT.md | Compliance score across 6 weighted dimensions |

### Round 2: Structured Debate

The Attacker presents findings. The Defender responds. The Auditor rules.

**Attacker findings use ATK-{N} IDs:**
```markdown
### ATK-1: Missing input validation on login form
**Severity:** HIGH
**Perspective:** Security Auditor
**File:** src/features/auth/LoginForm.tsx:45
**Evidence:** No sanitization of email input before API call
**Impact:** Potential XSS via reflected input
```

**Defender responses:**
- **Concede** — "ATK-1: Concede. Input validation was overlooked."
- **Dispute** (with evidence) — "ATK-1: Dispute. Validation exists at DEF-3 (src/lib/validators.ts:12)"
- **Mitigate** — "ATK-1: Mitigate. Server-side validation catches this, but client-side should be added."

**Auditor rulings:**
- **SUSTAINED** — Finding stands, becomes a gap
- **OVERRULED** — Evidence disproves the finding
- **SPLIT** — Requires Round 3 consensus

### Round 3: Consensus (If Needed)

Only for SPLIT rulings. The orchestrator makes the final call based on evidence from both sides.

### Consensus Detection

When all three agents independently flag the same issue, it's a **consensus failure** — the strongest possible signal. These are auto-confirmed without debate.

## Verdict Logic

| Verdict | Conditions |
|---------|-----------|
| **PASS** | Defender score 90%+, Attacker: 0 critical + 2 or fewer high (post-debate), Auditor score 90%+ |
| **CONDITIONAL_PASS** | Defender score 70%+, Attacker: 0 critical (post-debate), Auditor score 70%+ |
| **FAIL** | Any role below thresholds, OR critical finding, OR unresolved consensus failure |

## VERIFICATION.md Output

```yaml
---
verdict: CONDITIONAL_PASS
defender_score: 85
attacker_findings: 7
attacker_findings_post_debate: 4
auditor_compliance: 82
debate_rounds: 2
consensus_findings: 1
gaps:
  - id: ATK-1
    title: Missing input validation
    severity: high
    file: src/features/auth/LoginForm.tsx:45
---
```

Followed by:
- Executive Summary
- Evidence Matrix (from Defender)
- Findings (post-debate, from Attacker)
- Compliance Score Breakdown (from Auditor)
- Debate Results (Round 2 outcomes)
- Gap Tasks (actionable items for gap-close)

## Gap Closure

### Bug Clustering

Related bugs get fixed together. A stub in component A and a missing import in component B might be the same root cause.

**Clustering rules:**
1. **Same-file bugs cluster together** — Fixer already has file context
2. **Same-domain bugs cluster if sharing state** — e.g., store + component using that store
3. **Cross-domain bugs get dedicated fixers** — Integration issues need broader context
4. **Consensus findings get priority** — Issues all 3 verifiers flagged

### Agent Type Selection

| Bug Type | Agent | Rationale |
|----------|-------|-----------|
| Build errors / compile failures | gsd-executor | Needs to write/fix code |
| Logic bugs / wrong behavior | gsd-debugger | Needs to investigate root cause |
| Missing features / stubs | gsd-executor | Needs to implement functionality |
| CSS / layout issues | gsd-executor | Needs to write UI code |
| Wiring / integration gaps | gsd-executor | Needs to connect components |
| Security vulnerabilities | gsd-executor | Needs to implement security patterns |
| Performance issues | gsd-debugger | Needs to profile and optimize |

### Ralph 3-Level Self-Verify

Every fixer MUST verify their own work at three levels:

**Level 1: Code Review** — Read changed files. No syntax errors, no logic errors, no TODOs left, imports correct.

**Level 2: Logical Walk-Through** — Trace execution path. Data flows correctly? Edge cases handled? Fix interacts correctly with surrounding code?

**Level 3: Runtime Verification** — Build passes, tests pass, gap-specific verification passes.

```markdown
## Ralph Self-Verify Results
| Level | Status | Details |
|-------|--------|---------|
| Code Review | PASS | No syntax errors, imports correct |
| Logical Walk-Through | PASS | Data flows from form → store → API correctly |
| Runtime | PASS | Build passes, login test passes |
```

### Retry Logic

- **Max retries:** 5
- **Retry 1-2:** Standard recluster and respawn
- **Retry 3+:** Escalation analysis — why are these gaps persisting? Wrong agent type? Wrong root cause? Needs human input?
- **Exhaustion:** Display persistent gaps with diagnosis and offer: full re-verify, manual intervention, or accept as CONDITIONAL_PASS

### Lead Integration Check

After all fixers complete:

1. **Conflict Check** — No fixers modified the same files
2. **Build Verify** — Everything still compiles
3. **Test Suite** — All tests still pass
4. **Spot Check** — Read key files, verify fixes look correct
5. **Update VERIFICATION.md** — Mark closed gaps
