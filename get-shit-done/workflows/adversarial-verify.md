<purpose>
Execute a 3-round adversarial verification with Defender, Attacker, and Auditor agents.
Round 1: Independent assessment (parallel). Round 2: Structured debate. Round 3: Consensus (if needed).
</purpose>

<context_budget>
Lead stays under 50% context. Enforced by:
1. **Context-by-reference** — verification context written to `.context-brief.md`
2. **Delegated verdict synthesis** — Task() subagent reads all reports and writes VERIFICATION.md; lead receives verdict line only
3. **Spawn brevity** — verifier prompts are role + file paths + protocol ref
4. **Protocols by reference** — finding IDs, debate protocol, verdict logic, VERIFICATION.md template live in `ultra-conventions.md`
</context_budget>

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
```bash
AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
AGENT_TEAMS_MODE=$( [ "$AGENT_TEAMS_ENV" = "1" ] && echo true || echo false )
```

Display: `Coordination: {Agent Teams (native) | Task() subagents}`

Both paths produce identical DEFENSE/ATTACK/AUDIT/DEBATE.md + VERIFICATION.md. Agent Teams replaces file-serialized debate with live messaging.
</step>

<step name="write_context_brief">
Write `{phase_dir}/.context-brief.md` with verification context:

```bash
ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null
ls "${PHASE_DIR}"/*-SUMMARY.md 2>/dev/null
```

Include: phase info, goal from ROADMAP.md, phase directory path.
Note: "Read PLAN.md, SUMMARY.md files in phase directory. Read CLAUDE.md, DOMAINS.md, REQUIREMENTS.md, gsd-ultra.json from project root. Examine actual source code in src/."
</step>

<step name="round_1_independent">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > ADVERSARIAL VERIFICATION — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Round 1: Independent Assessment (parallel)
  > Defender  ({DEFENDER_MODEL})
  > Attacker  ({ATTACKER_MODEL})
  > Auditor   ({AUDITOR_MODEL})
Coordination: {Agent Teams (native) | Task() subagents}
```

**Verifier spawn spec:**

| Role | Agent Def | Output File | ID Format | Model |
|------|-----------|-------------|-----------|-------|
| Defender | ultra-defender | DEFENSE.md | DEF-{N} | {DEFENDER_MODEL} |
| Attacker | ultra-attacker | ATTACK.md | ATK-{N} | {ATTACKER_MODEL} |
| Auditor | ultra-auditor | AUDIT.md | compliance score | {AUDITOR_MODEL} |

**Prompt template (all 3 roles):**
```
Read the {agent_def} agent definition.
You are the {role} for Phase {phase}: {phase_name}.
Read context brief: {phase_dir}/.context-brief.md
Read ultra-conventions.md for finding IDs, debate protocol, verdict logic.
Write {output_file} to: {phase_dir}/
{role_specific_round1_instructions}
```

**If AGENT_TEAMS_MODE=true:** Spawn as persistent Teammate() (they persist through all rounds). Include Round 2 debate instructions in spawn prompt. Lead enters delegate mode.

**If AGENT_TEAMS_MODE=false:** Spawn as Task(subagent_type="general-purpose") in parallel. Wait for all 3 to complete.

**Verify Round 1 outputs:**
```bash
for f in DEFENSE ATTACK AUDIT; do
  [ -f "${PHASE_DIR}/${f}.md" ] && echo "ok ${f}.md" || echo "MISSING ${f}.md"
done
```
</step>

<step name="consensus_detection">
Before debate, check for consensus findings — issues flagged by all 3 roles.

Lead reads only the finding IDs/titles from each report (not full content). Cross-reference to find issues all 3 mention.

Consensus findings skip debate — they're automatically confirmed.
</step>

<step name="round_2_debate">
Display: `Round 2: Structured Debate — {N} findings to debate`

Only run if Attacker has HIGH or CRITICAL findings that aren't consensus findings.

**If AGENT_TEAMS_MODE=true:** Lead messages Attacker to begin debate. Teammates exchange findings via native messaging. Auditor messages lead with each ruling (SUSTAINED/OVERRULED/SPLIT). Auditor writes DEBATE.md after all findings debated.

**If AGENT_TEAMS_MODE=false:** Spawn debate moderator Task():
```
Task(
  prompt="You are the debate moderator for Phase {phase}.
  Read DEFENSE.md, ATTACK.md, AUDIT.md from {phase_dir}/.
  Read ultra-conventions.md for debate protocol and ruling format.
  Run structured debate for HIGH/CRITICAL non-consensus findings.
  Write DEBATE.md to: {phase_dir}/",
  subagent_type="general-purpose",
  model="{ATTACKER_MODEL}",
  description="Debate Phase {phase}"
)
```
</step>

<step name="round_3_consensus">
Only if Round 2 has SPLIT rulings. The orchestrator resolves based on:
1. Weight of evidence (file:line references vs. assertions)
2. Severity precedent (safety > style)
3. Consensus overlap (2 of 3 agree wins)

Most phases won't need Round 3.
</step>

<step name="delegate_verdict_synthesis">
Display: `Synthesizing verdict...`

Spawn a verdict synthesis subagent — lead does NOT read the full reports:

```
Task(
  prompt="Synthesize the adversarial verification verdict for Phase {phase}: {phase_name}.
  Read from {phase_dir}/: DEFENSE.md, ATTACK.md, AUDIT.md, DEBATE.md (if exists).
  Read ultra-conventions.md for VERIFICATION.md Template and Verdict Thresholds.
  Read context brief: {phase_dir}/.context-brief.md
  Write VERIFICATION.md to: {phase_dir}/{padded_phase}-VERIFICATION.md
  Return ONE line: VERDICT={PASS|CONDITIONAL_PASS|FAIL} score={N}/{M} gaps={N}",
  subagent_type="general-purpose",
  description="Synthesize verdict Phase {phase}"
)
```

Lead receives only the verdict line. VERIFICATION.md is written by the subagent.
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
 ULTRA > VERIFICATION PASSED
Phase {X}: {Name} — PASS
 > Next: /ultra:retrospective {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**If CONDITIONAL_PASS:**
```
Phase {X}: {Name} — CONDITIONAL_PASS
Options: 1. Accept > /ultra:retrospective {X}  2. /ultra:gap-close {X}
```

**If FAIL:**
```
Phase {X}: {Name} — FAIL
 > Next: /ultra:gap-close {X}
   /clear first for fresh context
```
</offer_next>
