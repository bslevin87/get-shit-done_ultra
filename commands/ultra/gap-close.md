---
name: ultra:gap-close
description: Read VERIFICATION.md gaps, spawn fixer agents, self-verify with retry
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
Close gaps found by adversarial verification. Reads VERIFICATION.md gaps, creates targeted fix plans, executes them, and self-verifies. Retries up to 2 times if gaps persist.

Context budget: ~20% orchestrator, 100% fresh per fixer agent.
</objective>

<execution_context>
@get-shit-done/workflows/execute-phase.md
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

## 1. Load Gaps

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init verify-work "${PHASE_ARG}")
```

Read VERIFICATION.md from phase directory. Parse `gaps:` from YAML frontmatter.

If no gaps found: "No gaps to close. Phase verification passed."

## 2. Create Fix Plans

For each gap, create a targeted fix plan:

```markdown
---
phase: {phase}
plan: {next_plan_number}
type: execute
wave: 1
depends_on: []
files_modified: [{gap artifact paths}]
autonomous: true
gap_closure: true
gap_source: "VERIFICATION.md"
---

<objective>
Close verification gap: {gap.truth}
Reason: {gap.reason}
</objective>

<tasks>
<task type="auto">
  <name>Fix: {gap.truth}</name>
  <files>{gap.artifacts[].path}</files>
  <action>
  {For each gap.missing item:}
  - {missing item}

  Gap reason: {gap.reason}
  Attacker finding: {relevant finding from ATTACK.md}
  </action>
  <verify>{verification command}</verify>
  <done>{gap.truth} is now achievable</done>
</task>
</tasks>
```

## 3. Execute Fix Plans

Spawn executor agents for each fix plan (same as execute-phase but with gap_closure flag).

Include file ownership from DOMAINS.md in executor prompts.

## 4. Self-Verify

After execution, run a quick verification:

```bash
# For each gap, check if the fix resolves it
for gap in gaps; do
  # Check artifact exists and isn't stub
  # Check wiring
  # Report status
done
```

## 5. Retry Logic

```
MAX_RETRIES=2
current_retry=0

while gaps_remain and current_retry < MAX_RETRIES:
  execute_fixes()
  self_verify()
  current_retry++

if gaps_remain:
  report remaining gaps
  suggest /ultra:adversarial-verify for full re-verification
```

## 6. Report

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► GAP CLOSE {COMPLETE|PARTIAL}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Gaps: {closed}/{total} closed
Retries: {N}/{MAX_RETRIES}

| Gap | Status | Fix |
|-----|--------|-----|
| {truth} | ✓ CLOSED / ✗ OPEN | {what was done} |

{If all closed:}
## ▶ Next: Re-verify

/ultra:adversarial-verify {X}

{If gaps remain:}
## ⚠ Remaining Gaps
{list of unclosed gaps with diagnosis}
```

</process>
