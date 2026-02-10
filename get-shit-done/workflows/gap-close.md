<purpose>
Close gaps found by adversarial verification. Reads VERIFICATION.md, clusters bugs intelligently, spawns fixer agents with the Ralph self-verification protocol, and retries up to 5 times.

This workflow combines bug clustering (group related fixes together) with Ralph self-verify (each fixer validates its own work at 3 levels before reporting success). The result: fewer fix iterations and higher first-pass fix rates.
</purpose>

<core_principle>
Fixing bugs in isolation fails. A stub in component A and a missing import in component B might be the same root cause (component A was never properly built). Bug clustering groups related issues so a single fixer can address the root cause, not just the symptoms.

Ralph self-verify prevents the #1 gap-close failure: "I fixed it" when the fix introduced a new bug or only addressed the surface issue.
</core_principle>

<bug_clustering>
**Clustering Rules:**

1. **Same-file bugs cluster together** — A fixer already has the file context
2. **Same-domain bugs cluster if they share state** — e.g., store + component using that store
3. **Cross-domain bugs get dedicated fixers** — Integration issues need broader context
4. **Consensus findings get priority clustering** — Issues all 3 verifiers flagged are treated as one root cause

```markdown
## Bug Clusters

### Cluster 1: {Domain} — {Root Cause Description}
**Bugs:** ATK-1, ATK-3, ATK-7
**Files:** src/features/auth/login.tsx, src/features/auth/store.ts
**Root cause:** Auth flow was partially implemented
**Agent type:** executor (missing features)

### Cluster 2: {Domain} — {Root Cause Description}
**Bugs:** ATK-5
**Files:** src/lib/api.ts
**Root cause:** Error handling missing in API client
**Agent type:** debugger (logic bug)
```
</bug_clustering>

<agent_type_selection>
**Choose the right agent for each cluster:**

| Bug Type | Agent | Rationale |
|----------|-------|-----------|
| Build errors / compile failures | gsd-executor | Needs to write/fix code, follows plan |
| Logic bugs / wrong behavior | gsd-debugger | Needs to investigate root cause |
| Missing features / stubs | gsd-executor | Needs to implement missing functionality |
| CSS / layout issues | gsd-executor | Needs to write UI code |
| Wiring / integration gaps | gsd-executor | Needs to connect components |
| Security vulnerabilities | gsd-executor | Needs to implement security patterns |
| Performance issues | gsd-debugger | Needs to profile and optimize |
</agent_type_selection>

<ralph_protocol>
**Ralph 3-Level Self-Verify**

Every fixer MUST run all 3 levels before reporting "fixed":

**Level 1: Code Review**
Read the changed files and verify:
- No syntax errors
- No obvious logic errors
- No TODOs or placeholders left
- Imports are correct

**Level 2: Logical Walk-Through**
Trace the execution path mentally:
- Does data flow from source to destination correctly?
- Are all edge cases handled?
- Does the fix interact correctly with surrounding code?

**Level 3: Runtime Verification**
Actually verify the fix works:
```bash
# Build passes
npm run build 2>/dev/null || npx tsc --noEmit 2>/dev/null

# Tests pass (if applicable)
npm test 2>/dev/null

# Specific verification command from the gap
{gap.verify_command}
```

**If any level fails:** The fixer must fix the issue before reporting, or report the failure honestly with diagnostics.
</ralph_protocol>

<fixer_spawn_template>
Every fixer prompt includes:

```markdown
<fix_context>
**Phase:** {phase_number} - {phase_name}
**Cluster:** {cluster_number} — {cluster_description}

**Bugs to Fix:**
{For each bug in cluster:}
- ATK-{N}: {finding title}
  Severity: {severity}
  File: {file}:{line}
  Evidence: {evidence summary}
  Impact: {impact}

**File Ownership:**
- OWN: {owned_paths}
- SHARED: {shared_paths}
- DO NOT TOUCH: {forbidden_paths}

**Ralph Self-Verify Protocol:**
After fixing, you MUST verify at 3 levels:
1. Code Review: Read your changes, check for errors
2. Logical Walk-Through: Trace execution path
3. Runtime: Build, test, verify specific fix

Report your Ralph results in the summary:
## Ralph Self-Verify Results
| Level | Status | Details |
|-------|--------|---------|
| Code Review | PASS/FAIL | {details} |
| Logical Walk-Through | PASS/FAIL | {details} |
| Runtime | PASS/FAIL | {details} |
</fix_context>
```
</fixer_spawn_template>

<process>

<step name="load_gaps" priority="first">
```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init verify-work "${PHASE_ARG}")
```

Read VERIFICATION.md from phase directory. Parse `gaps:` from YAML frontmatter.
Also read DEBATE.md if it exists (for post-debate severity adjustments).
Also read ATTACK.md for detailed evidence on each ATK-{N} finding.

If no gaps found: "No gaps to close. Phase verification passed."

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► GAP CLOSE — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Loading {N} gaps from VERIFICATION.md...
```
</step>

<step name="cluster_bugs">
Group gaps into clusters using clustering rules:

1. Group by file (same-file bugs together)
2. Group by domain (same-domain bugs sharing state)
3. Separate cross-domain issues
4. Prioritize consensus findings

Display:
```
◆ Clustered into {N} fix groups:
  → Cluster 1: {domain} — {description} ({M} bugs, {agent_type})
  → Cluster 2: {domain} — {description} ({M} bugs, {agent_type})
```
</step>

<step name="create_fix_plans">
For each cluster, create a targeted fix plan:

```markdown
---
phase: {phase}
plan: {next_plan_number}
type: execute
wave: 1
depends_on: []
files_modified: [{cluster artifact paths}]
autonomous: true
gap_closure: true
gap_source: "VERIFICATION.md"
cluster: {cluster_number}
bugs: [{ATK-N IDs}]
---

<objective>
Close verification gaps for cluster {N}: {description}

Bugs: {ATK-N list with titles}
Root cause: {identified root cause}

FILE OWNERSHIP:
- OWN: {owned_paths}
- DO NOT TOUCH: {forbidden_paths}
</objective>

<tasks>
<task type="auto">
  <name>Fix: {cluster description}</name>
  <files>{cluster file list}</files>
  <action>
  {For each gap in cluster:}
  - ATK-{N}: {what to fix, with evidence reference}

  Root cause: {root cause analysis}
  </action>
  <verify>
  Ralph Level 3:
  - Build: npm run build or npx tsc --noEmit
  - Test: npm test (if applicable)
  - Specific: {gap-specific verification}
  </verify>
  <done>{cluster bugs resolved, ralph self-verify passes all 3 levels}</done>
</task>
</tasks>
```
</step>

<step name="spawn_fixers">
Display:
```
◆ Spawning {N} fixers...
  → Cluster 1: {agent_type} ({model}) — {bug_count} bugs
  → Cluster 2: {agent_type} ({model}) — {bug_count} bugs
```

For each cluster, spawn a fixer with the appropriate agent type:

```
Task(
  prompt="Read the {agent_type} agent definition.
  Fix the following bugs for Phase {phase}: {phase_name}.

  {fixer_spawn_template with cluster details}

  Write your fix plan and execute it.
  Write summary to: {phase_dir}/{plan_id}-SUMMARY.md

  CRITICAL: Run Ralph 3-Level Self-Verify before reporting done.",
  subagent_type="{agent_type}",
  model="{EXECUTOR_MODEL}",
  description="Fix cluster {N} Phase {phase}"
)
```

Independent clusters run in parallel. Dependent clusters run sequentially.
</step>

<step name="lead_integration_check">
After all fixers complete:

Display:
```
◆ Lead integration check...
```

1. **Conflict Check** — verify no fixers modified the same files
   ```bash
   # Check for file conflicts between fix summaries
   ```
2. **Build Verify** — everything still compiles
   ```bash
   npm run build 2>/dev/null || npx tsc --noEmit 2>/dev/null
   ```
3. **Test Suite** — all tests still pass
   ```bash
   npm test 2>/dev/null
   ```
4. **Spot Check** — read key files to verify fixes look correct
5. **Update VERIFICATION.md** — mark closed gaps, update findings
</step>

<step name="retry_logic">
```
MAX_RETRIES=5
current_retry=0

while gaps_remain and current_retry < MAX_RETRIES:
  cluster_remaining_gaps()
  spawn_fixers_for_remaining()
  lead_integration_check()
  current_retry++

  if current_retry >= 3:
    # Escalation analysis: why are these gaps persisting?
    analyze_persistent_gaps()
    # Consider: wrong agent type? wrong root cause? needs human input?
```

On retry exhaustion (5 attempts, gaps still open):
```
◆ ESCALATION: {N} gaps remain after 5 attempts

Persistent gaps:
- ATK-{N}: {title} — failed {M} fix attempts
  Analysis: {why fixes didn't stick}
  Suggestion: {what might work — different approach, human intervention}

Options:
1. /ultra:adversarial-verify {X} — full re-verification with fresh eyes
2. Manual intervention — fix remaining gaps yourself
3. Accept as CONDITIONAL_PASS — document known limitations
```
</step>

<step name="commit">
```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit \
  "fix(ultra): gap-close for phase ${PHASE} — ${CLOSED}/${TOTAL} gaps closed" \
  --files ${FIX_SUMMARY_FILES}
```
</step>

</process>

<offer_next>
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► GAP CLOSE {COMPLETE|PARTIAL}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Gaps: {closed}/{total} closed
Retries: {N}/{MAX_RETRIES}
Clusters: {N} clusters processed

| Cluster | Bugs | Agent | Ralph | Status |
|---------|------|-------|-------|--------|
| 1 | ATK-1,3,7 | executor | 3/3 PASS | ✓ CLOSED |
| 2 | ATK-5 | debugger | 3/3 PASS | ✓ CLOSED |

{If all closed:}
───────────────────────────────────────────────────────

## ▶ Next: Re-verify

/ultra:adversarial-verify {X}

<sub>/clear first → fresh context window</sub>

{If gaps remain:}
## ⚠ Remaining Gaps
{list of unclosed gaps with diagnosis and escalation analysis}
```
</offer_next>

<success_criteria>
- [ ] Gaps loaded from VERIFICATION.md
- [ ] Bugs clustered intelligently (file, domain, cross-domain)
- [ ] Agent types selected per cluster
- [ ] Fixers spawned with Ralph self-verify protocol in prompts
- [ ] Ralph 3-level self-verify executed by each fixer
- [ ] Lead integration check passed (conflict, build, test, spot check)
- [ ] Retry logic with up to 5 attempts
- [ ] Escalation analysis on persistent gaps
- [ ] Fix summaries committed
- [ ] User knows next step (re-verify or escalate)
</success_criteria>
