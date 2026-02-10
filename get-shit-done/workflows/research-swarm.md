<purpose>
Execute a 4-perspective research swarm for a phase. Spawns 4 specialized researcher agents in parallel, collects their outputs, and synthesizes into a single RESEARCH.md.

Replaces GSD's single-researcher model with multi-perspective research: Pattern Analyst, Domain Expert, Risk Analyst, and UX Investigator each contribute a unique lens.
</purpose>

<core_principle>
Four perspectives are better than one. Each researcher has a different "attack surface" — patterns, domain expertise, risks, and UX. The synthesis combines them into a single research document that captures insights no single researcher would find alone.
</core_principle>

<relay_protocol>
**RELAY: Cross-Researcher Discovery Notification**

When a researcher discovers information that affects another researcher's domain, they include a RELAY section at the end of their output:

```markdown
## RELAY (Cross-Perspective Discoveries)

### RELAY → Risk Analyst
**Discovery:** The auth library we recommend (jose) had a CVE in v4.x
**Impact:** Risk assessment should flag JWT implementation as requiring v5+ pinning
**Source:** npm audit / GitHub advisories

### RELAY → UX Investigator
**Discovery:** The API returns paginated results with cursor-based pagination
**Impact:** UX should plan for infinite scroll or load-more patterns, not page numbers
**Source:** API documentation review
```

**Rules:**
- Researchers write RELAY sections — they don't read each other's outputs directly
- The synthesizer reads all RELAY sections and weaves cross-discoveries into the synthesis
- RELAYs are informational, not prescriptive — the target researcher may already know
- Keep RELAYs focused: only flag discoveries the other researcher likely wouldn't find independently
</relay_protocol>

<scaling_guidance>
**How many researchers to spawn:**

| Phase Size | Files | Researchers | Rationale |
|------------|-------|-------------|-----------|
| Small | 2-3 files | 2 (Pattern + Domain) | UX and Risk perspectives add noise for trivial phases |
| Medium | 4-8 files | 3 (Pattern + Domain + Risk) | Most phases; UX added only if phase has UI components |
| Large | 10+ files | 4 (all perspectives) | Full swarm for greenfield or complex phases |
| Greenfield | New project | 4 (all perspectives) | Always full swarm for new codebases |

**Decision logic:**
```
if phase has UI components:
  include UX Investigator
if phase modifies >3 files or touches auth/data/payments:
  include Risk Analyst
always include Pattern Analyst + Domain Expert
```

When spawning fewer than 4, adjust the synthesis to note which perspectives were included and which were skipped (with rationale).
</scaling_guidance>

<process>

<step name="initialize" priority="first">
Load all context in one call:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init plan-phase "${PHASE_ARG}" --include state,roadmap,requirements,context)
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `phase_slug`, `padded_phase`.

Also read Ultra config:
```bash
ULTRA_CONFIG=$(cat gsd-ultra.json 2>/dev/null || echo '{}')
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_profile || 'balanced'")
```

Resolve models for each researcher role from gsd-ultra.json:
```bash
# Parse role models from Ultra config based on profile
PATTERN_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.pattern_analyst['$MODEL_PROFILE']")
DOMAIN_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.domain_expert['$MODEL_PROFILE']")
RISK_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.risk_analyst['$MODEL_PROFILE']")
UX_MODEL=$(echo "$ULTRA_CONFIG" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).model_routing.roles.ux_investigator['$MODEL_PROFILE']")
```

If gsd-ultra.json missing, fall back to gsd-tools.js model resolution:
```bash
PATTERN_MODEL=$(node ~/.claude/get-shit-done/bin/gsd-tools.js resolve-model gsd-phase-researcher)
# Use same model for all 4 as fallback
```

**Determine researcher count** using scaling guidance:
- Check phase scope (files count, UI involvement, sensitivity)
- Default: 4 researchers for medium+ phases
- Minimum: 2 researchers (Pattern + Domain) for small phases
</step>

<step name="prepare_research_dir">
```bash
mkdir -p "${PHASE_DIR}/research"
```
</step>

<step name="build_research_prompt">
Construct shared context for all researchers:

```bash
STATE_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).state_content || ''")
ROADMAP_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).roadmap_content || ''")
CONTEXT_CONTENT=$(echo "$INIT" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).context_content || ''")
PHASE_DESC=$(node ~/.claude/get-shit-done/bin/gsd-tools.js roadmap get-phase "${PHASE_ARG}" | node -pe "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).section")
```

Shared context block (passed to all researchers):
```markdown
<phase_context>
**Phase:** {phase_number} - {phase_name}
**Goal:** {phase goal from roadmap}
**Description:** {phase_desc}

**User Decisions (from CONTEXT.md):**
{context_content}

**Project State:**
{state_content}
</phase_context>

<relay_instructions>
If you discover information that would be valuable to another researcher's perspective,
include a RELAY section at the end of your output:

## RELAY (Cross-Perspective Discoveries)
### RELAY → {Target Perspective}
**Discovery:** {what you found}
**Impact:** {how it affects their research}
**Source:** {where you found it}

Only relay genuinely cross-cutting discoveries — things the other researcher
wouldn't find independently from their perspective.
</relay_instructions>
```
</step>

<step name="spawn_researchers">
Display banner:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► RESEARCH SWARM — PHASE {X}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

◆ Spawning {N} researchers in parallel...
  → Pattern Analyst   ({PATTERN_MODEL})
  → Domain Expert     ({DOMAIN_MODEL})
  → Risk Analyst      ({RISK_MODEL})     {if included}
  → UX Investigator   ({UX_MODEL})       {if included}
```

Spawn researchers in parallel using Task():

```
Task(
  prompt="First, read the ultra-pattern-analyst agent definition.
  Then research patterns for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/pattern-analysis.md",
  subagent_type="general-purpose",
  model="{PATTERN_MODEL}",
  description="Research patterns Phase {phase}"
)

Task(
  prompt="First, read the ultra-domain-expert agent definition.
  Then research domain expertise for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/domain-expertise.md",
  subagent_type="general-purpose",
  model="{DOMAIN_MODEL}",
  description="Research domain Phase {phase}"
)

Task(
  prompt="First, read the ultra-risk-analyst agent definition.
  Then research risks for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/risk-analysis.md",
  subagent_type="general-purpose",
  model="{RISK_MODEL}",
  description="Research risks Phase {phase}"
)

Task(
  prompt="First, read the ultra-ux-investigator agent definition.
  Then research UX for Phase {phase}: {phase_name}.
  {shared_context}
  Write to: {phase_dir}/research/ux-investigation.md",
  subagent_type="general-purpose",
  model="{UX_MODEL}",
  description="Research UX Phase {phase}"
)
```

All spawn in parallel. Wait for all to complete.
</step>

<step name="verify_outputs">
Verify all perspective files were written:

```bash
for f in pattern-analysis domain-expertise risk-analysis ux-investigation; do
  [ -f "${PHASE_DIR}/research/${f}.md" ] && echo "✓ ${f}.md" || echo "✗ ${f}.md MISSING"
done
```

If any missing: report which researcher failed, offer retry or continue with available perspectives.
</step>

<step name="synthesize">
Display:
```
◆ Synthesizing {N} perspectives into RESEARCH.md...
```

Read all perspective files. The synthesizer has 3 jobs:

**Job 1: Cross-Perspective Synthesis**
Combine findings into a coherent research document.

**Job 2: RELAY Integration**
Read RELAY sections from each researcher. Weave cross-discoveries into the relevant sections of the synthesis. Note which RELAYs led to new insights vs. were already covered.

**Job 3: Conflict Resolution**
When researchers disagree (e.g., Domain Expert recommends library A, Risk Analyst flags library A as risky):

```markdown
## Conflicts Identified

### Conflict 1: {Topic}
**Perspective A ({researcher}):** {position}
**Perspective B ({researcher}):** {position}
**Resolution:** {resolved | unresolved}
**Rationale:** {if resolved, why this position wins}
**Action needed:** {if unresolved, flag for human review}
```

Conflict resolution rules:
1. **Safety > convenience** — Risk Analyst's concerns override Domain Expert's recommendations
2. **Evidence > opinion** — the perspective with more concrete evidence wins
3. **Flag when uncertain** — if neither side has clear evidence, mark as unresolved for human review
4. **Document all conflicts** — even resolved ones, so the planner understands trade-offs

Write synthesized RESEARCH.md:

```markdown
# Phase {X}: {Name} — Research (Ultra {N}-Perspective)

**Researched:** {timestamp}
**Perspectives:** {N} ({list of perspectives used})
**Perspectives skipped:** {list and rationale, if any}
**RELAYs processed:** {count}
**Conflicts identified:** {count resolved} resolved, {count unresolved} unresolved

## Synthesis Summary
{2-3 paragraphs combining key findings from all perspectives}

## User Constraints
{from CONTEXT.md if exists — copied verbatim}

## Standard Stack
{from domain-expertise.md}

## Architecture Patterns
{from pattern-analysis.md + domain-expertise.md}

## Code Patterns
{from pattern-analysis.md}

## Risk Assessment
{from risk-analysis.md — critical/high risks}

## Edge Cases
{from risk-analysis.md}

## UX Requirements
{from ux-investigation.md — states, accessibility, responsive}

## Don't Hand-Roll
{from domain-expertise.md}

## Common Pitfalls
{merged from all perspectives}

## Cross-Perspective Insights
{findings that emerged from combining perspectives — things no single researcher would have caught}
{Include RELAY-derived insights here}

## Conflicts and Resolutions
{from conflict resolution step — resolved and unresolved}

## Open Questions
{gaps from all perspectives + unresolved conflicts needing human input}

## Sources
{consolidated source list}
```

Write to: `${PHASE_DIR}/${PADDED_PHASE}-RESEARCH.md`
</step>

<step name="commit">
```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(ultra): research swarm for phase ${PHASE}" --files "${PHASE_DIR}/research/pattern-analysis.md" "${PHASE_DIR}/research/domain-expertise.md" "${PHASE_DIR}/research/risk-analysis.md" "${PHASE_DIR}/research/ux-investigation.md" "${PHASE_DIR}/${PADDED_PHASE}-RESEARCH.md"
```
</step>

</process>

<offer_next>
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 ULTRA ► RESEARCH SWARM COMPLETE ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Phase {X}: {Name}** — {N} perspectives synthesized

| Perspective | Confidence | Key Finding | RELAYs Sent |
|-------------|------------|-------------|-------------|
| Pattern Analyst | {level} | {1-line} | {count} |
| Domain Expert | {level} | {1-line} | {count} |
| Risk Analyst | {level} | {1-line} | {count} |
| UX Investigator | {level} | {1-line} | {count} |

Conflicts: {N resolved}, {M unresolved}
Research: {phase_dir}/{padded_phase}-RESEARCH.md

───────────────────────────────────────────────────────

## ▶ Next Up

**Adversarial Planning** — Builder/Critic debate loop

/ultra:adversarial-plan {X}

<sub>/clear first → fresh context window</sub>
```
</offer_next>

<success_criteria>
- [ ] Researcher count determined from scaling guidance
- [ ] All researcher agents spawned in parallel
- [ ] All perspective files written to research/ directory
- [ ] RELAY sections processed and integrated
- [ ] Conflicts between researchers identified and resolved (or flagged)
- [ ] Perspectives synthesized into single RESEARCH.md
- [ ] Cross-perspective insights identified
- [ ] RESEARCH.md committed to git
- [ ] User knows next step (adversarial-plan)
</success_criteria>
