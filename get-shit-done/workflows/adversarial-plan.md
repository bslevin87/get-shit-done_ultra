<purpose>
Execute a Builder/Critic adversarial debate loop to create high-quality phase plans. Loop continues until Critic signals APPROVED (zero BLOCKERs) or max rounds (5).
</purpose>

<context_budget>
Lead stays under 50% context. Enforced by:
1. **Context-by-reference** — shared context written to `.context-brief.md`, not embedded in spawn prompts
2. **Spawn brevity** — Builder/Critic prompts are role + file paths + protocol ref
3. **Protocols by reference** — severity levels, convergence, auction rules live in `ultra-conventions.md`
</context_budget>

<process>

<step name="initialize" priority="first">
Load all context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init plan-phase "${PHASE_ARG}" --include state,roadmap,requirements,context,research)
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `padded_phase`, `has_research`, `has_context`.

Resolve models for Builder and Critic:
```bash
ULTRA_CONFIG=$(cat gsd-ultra.json 2>/dev/null || echo '{}')
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_profile || 'balanced'")

BUILDER_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.builder['$MODEL_PROFILE']")
CRITIC_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.critic['$MODEL_PROFILE']")
```

Fallback: both use `gsd-planner` model from gsd-tools.js.

Set MAX_ROUNDS=5, current_round=1.
</step>

<step name="detect_agent_teams" priority="after-init">
```bash
AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
AGENT_TEAMS_MODE=$( [ "$AGENT_TEAMS_ENV" = "1" ] && echo true || echo false )
```

Display: `Coordination: {Agent Teams (native) | Task() subagents}`

Both paths produce identical PLAN.md files and approval flow.
</step>

<step name="check_research">
If RESEARCH.md missing and research is enabled:
```
Warning: No RESEARCH.md found. Run /ultra:research-swarm {phase} first for best results.
Proceeding without research...
```
</step>

<step name="write_context_brief">
Write `{phase_dir}/.context-brief.md` with planning context (format in `ultra-conventions.md`):

```bash
STATE_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).state_content || ''")
ROADMAP_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).roadmap_content || ''")
REQUIREMENTS_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).requirements_content || ''")
CONTEXT_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).context_content || ''")
RESEARCH_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).research_content || ''")
```

Include: phase info, roadmap, requirements, user decisions, research summary (first 5 lines of RESEARCH.md).
Also note: "Read CLAUDE.md, DOMAINS.md, gsd-ultra.json from project root."
</step>

<step name="debate_loop">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > ADVERSARIAL PLANNING — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Coordination: {Agent Teams (native) | Task() subagents}
```

**Debate spawn spec:**

| Role | Agent Def | Model | Job |
|------|-----------|-------|-----|
| Builder | ultra-builder | {BUILDER_MODEL} | Create/revise PLAN.md files in {phase_dir}/ |
| Critic | ultra-critic | {CRITIC_MODEL} | Review with BLOCKER/WARNING/SUGGESTION; signal APPROVED when 0 BLOCKERs |

**Prompt template for both roles:**
```
Read the {agent_def} agent definition.
You are the {role} for Phase {phase}: {phase_name}.
Read context brief: {phase_dir}/.context-brief.md
Read ultra-conventions.md for severity levels, convergence, max rounds.
Read CLAUDE.md, DOMAINS.md, gsd-ultra.json from project root.
{role_specific_instructions}
```

**If AGENT_TEAMS_MODE=true:** Spawn Builder and Critic as persistent Teammate(). Debate happens via native messaging. Lead monitors round count and intervenes at MAX_ROUNDS.

**If AGENT_TEAMS_MODE=false:** Sequential Task() calls:
1. Builder creates plans -> lead reads plan file list (not content)
2. Critic reviews plans (reads files directly) -> returns APPROVED or REVISE with feedback
3. If REVISE: Builder revises with Critic feedback injected -> loop back to Critic
4. Increment current_round each loop

**Convergence:** Critic returns "APPROVED" (0 BLOCKERs). See `ultra-conventions.md` for severity levels.

**Max rounds handling (both modes):**
```
Warning: Max rounds ({MAX_ROUNDS}) reached. {N} BLOCKERs remain.
Options: 1. Force proceed  2. Provide guidance  3. Abort
```
Wait for user input.
</step>

<step name="finalize">
Read final plan file list:
```bash
PLAN_COUNT=$(ls "${PHASE_DIR}"/*-PLAN.md 2>/dev/null | wc -l)
```

Commit:
```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(ultra): adversarial plan for phase ${PHASE} — approved round ${ROUND}" --files "${PHASE_DIR}"/*-PLAN.md
```

Update ROADMAP.md with plan count and objectives.
</step>

</process>

<offer_next>
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA > ADVERSARIAL PLANNING COMPLETE

Phase {X}: {Name} — {N} plans, approved round {R}/{MAX}
Debate: {rounds} rounds, {total_blockers} blockers resolved

 > Next: /ultra:parallel-execute {X}
   /clear first for fresh context
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</offer_next>
