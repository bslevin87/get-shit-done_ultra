<purpose>
Execute the knowledge flywheel for a completed phase. Reads all phase artifacts, generates a RETROSPECTIVE.md, extracts conventions for CLAUDE.md, logs architectural decisions to STATE.md, and updates DOMAINS.md if boundaries shifted.

The flywheel is Ultra's compound learning mechanism. Each retrospective makes subsequent phases faster and higher quality because researchers read CLAUDE.md conventions, planners read STATE.md decisions, and verifiers check against accumulated standards.
</purpose>

<core_principle>
Knowledge compounds. Phase 1's retrospective teaches Phase 2's research swarm what patterns to look for. By Phase 8, research is 60% faster because conventions are documented, verification finds 75% fewer issues because patterns are established, and planning produces better first drafts because decisions are logged.

The flywheel effect: Learn → Document → Apply → Verify → Learn more.
</core_principle>

<process>

<step name="initialize" priority="first">
Load all context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init phase-op "${PHASE_ARG}" --include state,roadmap)
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `padded_phase`.

Read all phase artifacts:
```bash
# Verification report (primary input)
VERIFICATION="${PHASE_DIR}/${PADDED_PHASE}-VERIFICATION.md"
# All summary files
SUMMARIES=$(ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null)
# All plan files
PLANS=$(ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null)
# Attack/Defense/Audit reports if they exist
DEFENSE="${PHASE_DIR}/DEFENSE.md"
ATTACK="${PHASE_DIR}/ATTACK.md"
AUDIT="${PHASE_DIR}/AUDIT.md"
# Research
RESEARCH="${PHASE_DIR}/${PADDED_PHASE}-RESEARCH.md"
```

Also read project-level files:
```bash
CLAUDE_MD=$(cat CLAUDE.md 2>/dev/null || echo '')
STATE_MD=$(cat .planning/STATE.md 2>/dev/null || echo '')
DOMAINS_MD=$(cat DOMAINS.md 2>/dev/null || echo '')
```

Count existing decisions for DEC-{NNN} numbering:
```bash
LAST_DEC=$(grep -oP 'DEC-\K\d+' .planning/STATE.md 2>/dev/null | sort -n | tail -1 || echo "0")
NEXT_DEC=$((LAST_DEC + 1))
```
</step>

<step name="generate_retrospective">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► KNOWLEDGE FLYWHEEL — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Analyzing phase artifacts...
  → VERIFICATION.md
  → {N} SUMMARY.md files
  → {M} PLAN.md files
  → DEFENSE.md / ATTACK.md / AUDIT.md
```

Read all artifacts. Generate RETROSPECTIVE.md:

```markdown
# Phase {X}: {Name} — Retrospective

**Date:** {timestamp}
**Verdict:** {from VERIFICATION.md}
**Plans:** {N} plans, {M} tasks
**Commits:** {count from git log}

## What Delivered

{Concrete list of what was built, from SUMMARY.md files}
- {artifact}: {description} ({file count} files, {line count} lines)

## What Went Well

{Analyze SUMMARY.md + DEFENSE.md for positive patterns}
- {pattern}: {why it worked, with evidence}

## What Went Wrong

{Analyze ATTACK.md + AUDIT.md + gap-close history for problems}
- {issue}: {what happened, root cause, how it was resolved}

## What We Learned

{Cross-reference all artifacts for insights}
- {lesson}: {how this changes future phases}

## Process Metrics

| Metric | Value | Benchmark |
|--------|-------|-----------|
| Plans created | {N} | — |
| Tasks executed | {M} | — |
| Verification verdict | {verdict} | PASS |
| Attacker findings (critical/high) | {N}/{M} | 0/≤2 |
| Auditor compliance | {pct}% | ≥90% |
| Gap-close rounds | {N} | 0-1 |
| Total commits | {N} | — |

## Patterns Identified

{Code patterns that emerged during this phase — candidates for CLAUDE.md}

### Pattern 1: {Name}
**Observed in:** {files}
**Description:** {what the pattern is}
**Convention suggestion:** {proposed CLAUDE.md addition}

### Pattern 2: {Name}
...

## Architectural Decisions Made

{Decisions made during this phase — candidates for STATE.md DEC-{NNN}}

### Decision 1: {Title}
**Context:** {why this decision was needed}
**Decision:** {what was decided}
**Rationale:** {why}
**Alternatives considered:** {what else was evaluated}

## Domain Boundary Changes

{If domain boundaries shifted during execution}
- {domain}: {what changed and why}

## Recommendations for Next Phase

{Based on everything learned}
- {recommendation}: {rationale}
```

Write to: `${PHASE_DIR}/RETROSPECTIVE.md`
</step>

<step name="skill_extraction">
Display:
```
◆ Extracting conventions for CLAUDE.md...
```

Analyze the completed code for 2-3 patterns worth codifying. For each:

1. Identify the pattern from actual code (not theoretical)
2. Draft a concise convention statement
3. Show the user with evidence

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 CONVENTION SUGGESTION 1 of {N}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Pattern: {name}
Evidence: {file:line examples}

Proposed CLAUDE.md addition:
───────────────────────────────────────────────────────
{convention text — 1-3 lines}
───────────────────────────────────────────────────────

Add this convention to CLAUDE.md?
```

Use AskUserQuestion for each convention:
- **Yes** — append to CLAUDE.md
- **No** — skip
- **Edit** — user provides modified text

Only add conventions the user approves. This keeps CLAUDE.md as the user's document, not auto-generated noise.
</step>

<step name="decision_logging">
Display:
```
◆ Logging architectural decisions to STATE.md...
```

For each architectural decision identified in the retrospective:

```markdown
### DEC-{NNN}: {Title}
**Phase:** {X} — {Name}
**Date:** {timestamp}
**Context:** {why}
**Decision:** {what}
**Rationale:** {why this over alternatives}
```

Append to the `## Decisions` section of STATE.md. If no such section exists, create it.

Use `node ~/.claude/get-shit-done/bin/gsd-tools.js state add-decision` where possible:
```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js state add-decision \
  --summary "DEC-${NEXT_DEC}: {title}" \
  --phase "${PHASE}" \
  --rationale "{rationale}"
```
</step>

<step name="domain_review">
If domain boundaries shifted during the phase (files created outside expected domains, shared files that should be owned, etc.):

Display:
```
◆ Reviewing domain boundaries...
```

Show changes to user:
```
Domain boundary changes detected:

1. {domain} expanded: added {path}
   Reason: {why}

2. {shared_file} should be owned by {domain}
   Reason: {why}

Update DOMAINS.md with these changes?
```

Use AskUserQuestion for approval. Only update if user agrees.
</step>

<step name="commit">
```bash
# Stage all retrospective artifacts
FILES="${PHASE_DIR}/RETROSPECTIVE.md"

# Include CLAUDE.md if conventions were added
[ -n "$CONVENTIONS_ADDED" ] && FILES="$FILES CLAUDE.md"

# Include STATE.md if decisions were logged
[ -n "$DECISIONS_LOGGED" ] && FILES="$FILES .planning/STATE.md"

# Include DOMAINS.md if boundaries changed
[ -n "$DOMAINS_UPDATED" ] && FILES="$FILES DOMAINS.md"

node ~/.claude/get-shit-done/bin/gsd-tools.js commit \
  "docs(ultra): retrospective for phase ${PHASE} — ${CONVENTIONS_COUNT} conventions, ${DECISIONS_COUNT} decisions" \
  --files $FILES
```
</step>

</process>

<offer_next>
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► KNOWLEDGE FLYWHEEL COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Phase {X}: {Name} — Retrospective captured

| Metric | Value |
|--------|-------|
| Conventions added to CLAUDE.md | {N} |
| Decisions logged (DEC-{start}..DEC-{end}) | {M} |
| Domain boundaries updated | {yes/no} |
| Patterns documented | {P} |

📈 Flywheel Effect
Research for Phase {X+1} will be faster — {N} new conventions
guide the research swarm. Verification will be stricter — {M}
decisions set the bar higher. Planning will be smarter — lessons
learned feed directly into the next adversarial-plan loop.

───────────────────────────────────────────────────────

## ▶ Next Phase

Ready to start Phase {X+1}:

/ultra:research-swarm {X+1}

<sub>/clear first → fresh context window</sub>
```
</offer_next>

<success_criteria>
- [ ] All phase artifacts read (VERIFICATION, SUMMARY, PLAN, DEFENSE, ATTACK, AUDIT)
- [ ] RETROSPECTIVE.md written with all sections
- [ ] 2-3 convention suggestions presented to user
- [ ] Approved conventions added to CLAUDE.md
- [ ] Architectural decisions logged as DEC-{NNN} in STATE.md
- [ ] DOMAINS.md updated if boundaries shifted (with user approval)
- [ ] All changes committed
- [ ] Flywheel metrics presented
- [ ] User knows next step
</success_criteria>
