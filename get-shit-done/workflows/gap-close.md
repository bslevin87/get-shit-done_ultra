<purpose>
Close gaps found by adversarial verification. Reads VERIFICATION.md, clusters bugs, spawns fixer agents with Ralph self-verification, and retries up to 5 times.
</purpose>

<context_budget>
Lead stays under 50% context. Enforced by:
1. **Protocols by reference** — bug clustering rules, agent type selection, Ralph protocol, fixer spawn template live in `ultra-conventions.md`
2. **Spawn brevity** — fixer prompts are cluster details + ownership + "see ultra-conventions.md for Ralph protocol"
3. **Context-by-reference** — gap details read from VERIFICATION.md by fixers, not embedded in prompts
</context_budget>

<process>

<step name="load_gaps" priority="first">
```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init verify-work "${PHASE_ARG}")
```

Read VERIFICATION.md from phase directory. Parse `gaps:` from YAML frontmatter.
Also read DEBATE.md if it exists (for post-debate severity adjustments).
Also read ATTACK.md for detailed evidence on each ATK-{N} finding.

If no gaps found: "No gaps to close. Phase verification passed."
</step>

<step name="detect_agent_teams" priority="after-init">
```bash
AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
AGENT_TEAMS_MODE=$( [ "$AGENT_TEAMS_ENV" = "1" ] && echo true || echo false )
```

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > GAP CLOSE — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Loading {N} gaps from VERIFICATION.md...
Coordination: {Agent Teams (native) | Task() subagents}
```

Both paths produce identical fix SUMMARY.md files and Ralph self-verify results.
</step>

<step name="cluster_bugs">
Group gaps into clusters using rules from `ultra-conventions.md` (Bug Clustering Rules):

1. Group by file (same-file bugs together)
2. Group by domain (same-domain bugs sharing state)
3. Separate cross-domain issues
4. Prioritize consensus findings

Select agent type per cluster using table from `ultra-conventions.md` (Agent Type Selection).

Display:
```
Clustered into {N} fix groups:
  > Cluster 1: {domain} — {description} ({M} bugs, {agent_type})
  > Cluster 2: {domain} — {description} ({M} bugs, {agent_type})
```
</step>

<step name="create_fix_plans">
For each cluster, create a targeted fix plan with YAML frontmatter:

```markdown
---
phase: {phase}
plan: {next_plan_number}
type: execute
wave: 1
autonomous: true
gap_closure: true
cluster: {cluster_number}
bugs: [{ATK-N IDs}]
---

<objective>
Close verification gaps for cluster {N}: {description}
Bugs: {ATK-N list}  |  Root cause: {identified root cause}
FILE OWNERSHIP: OWN: {paths} | DO NOT TOUCH: {paths}
</objective>

<tasks>
<task type="auto">
  <name>Fix: {cluster description}</name>
  <files>{cluster file list}</files>
  <action>{fix instructions per ATK-N}</action>
  <verify>Ralph Level 3 (see ultra-conventions.md)</verify>
  <done>{cluster bugs resolved, Ralph passes all 3 levels}</done>
</task>
</tasks>
```
</step>

<step name="spawn_fixers">
Display:
```
Spawning {N} fixers...
  > Cluster 1: {agent_type} ({model}) — {bug_count} bugs
  > Cluster 2: {agent_type} ({model}) — {bug_count} bugs
```

**Fixer prompt template:**
```
Read the {agent_type} agent definition.
Fix bugs for Phase {phase}: {phase_name}.
Read fix plan at: {plan_path}
Read VERIFICATION.md and ATTACK.md from {phase_dir}/ for evidence.
Read ultra-conventions.md for Ralph Self-Verify Protocol and Fixer Spawn Template.
{file_ownership_block}
Write summary to: {phase_dir}/{plan_id}-SUMMARY.md
CRITICAL: Run Ralph 3-Level Self-Verify before reporting done.
```

**If AGENT_TEAMS_MODE=true:** Lead creates shared task list from clusters. Spawn fixer Teammate()s who self-claim. Fixers message lead with Ralph results. TaskCompleted requires Ralph PASS.

**If AGENT_TEAMS_MODE=false:** Task(subagent_type="{agent_type}") per cluster. Independent clusters run in parallel. Dependent clusters run sequentially.
</step>

<step name="lead_integration_check">
After all fixers complete:

1. **Conflict Check** — verify no fixers modified the same files
2. **Build Verify** — `npm run build 2>/dev/null || npx tsc --noEmit 2>/dev/null`
3. **Test Suite** — `npm test 2>/dev/null`
4. **Spot Check** — read key files to verify fixes look correct
5. **Update VERIFICATION.md** — mark closed gaps, update findings
</step>

<step name="retry_logic">
```
MAX_RETRIES=5, current_retry=0

while gaps_remain and current_retry < MAX_RETRIES:
  cluster_remaining_gaps() -> spawn_fixers() -> lead_integration_check()
  current_retry++
  if current_retry >= 3: analyze_persistent_gaps()
```

On exhaustion (5 attempts, gaps still open):
```
ESCALATION: {N} gaps remain after 5 attempts
Options: 1. /ultra:adversarial-verify {X}  2. Manual fix  3. Accept as CONDITIONAL_PASS
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
 ULTRA > GAP CLOSE {COMPLETE|PARTIAL}

Gaps: {closed}/{total} closed | Retries: {N}/{MAX}

{If all closed:}
 > Next: /ultra:adversarial-verify {X}
   /clear first for fresh context

{If gaps remain:}
 Warning: {N} gaps remain — see escalation options above
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</offer_next>
