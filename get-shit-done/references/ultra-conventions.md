# Ultra Conventions Reference

Ultra extends GSD with multi-perspective research, adversarial planning, adversarial verification, and compound learning. These conventions define how Ultra components interact.

## Pipeline Stages

| Stage | Command | Agents | Output |
|-------|---------|--------|--------|
| 1. Research Swarm | `/ultra:research-swarm` | Pattern Analyst, Domain Expert, Risk Analyst, UX Investigator | 4 perspective files + RESEARCH.md |
| 2. Adversarial Plan | `/ultra:adversarial-plan` | Builder, Critic | PLAN.md files (approved) |
| 3. Parallel Execute | `/ultra:parallel-execute` | GSD Executors (with file ownership) | SUMMARY.md files |
| 4. Adversarial Verify | `/ultra:adversarial-verify` | Defender, Attacker, Auditor | DEFENSE/ATTACK/AUDIT/DEBATE.md + VERIFICATION.md |
| 5. Gap Close | `/ultra:gap-close` | GSD Executors + Debuggers | Gap fix commits |
| 6. Retrospective | `/ultra:retrospective` | Orchestrator (interactive) | RETROSPECTIVE.md + CLAUDE.md + STATE.md updates |

## Context Budget Protocol

Every Ultra workflow enforces a context budget to keep the lead agent under 50% context window utilization.

### Principles

1. **Context-by-Reference** — Write shared context to `{phase_dir}/.context-brief.md` once. Spawn prompts reference the file path instead of embedding context inline. Eliminates N-times duplication.
2. **Delegated Synthesis** — Large file reads (research perspectives, verification reports) happen in a Task() subagent, not the lead. The subagent writes the output file and returns a 3-line summary. Lead never reads the raw files.
3. **Message Brevity** — Spawn prompts contain role + file paths + protocol reference. Bulk context lives in `.context-brief.md` and `ultra-conventions.md`, not in the prompt.
4. **Spawn Budget** — Each spawn prompt should be <200 tokens of unique content (role, paths, specific instructions). Shared context comes from file references.

### `.context-brief.md` Format

```markdown
# Context Brief — Phase {X}: {Name}

**Phase:** {phase_number} - {phase_name}
**Goal:** {phase goal from roadmap}
**Description:** {phase description}

## User Decisions
{from CONTEXT.md}

## Project State
{from STATE.md — key decisions only}

## Research Summary
{from RESEARCH.md header — 3-5 lines, if exists}

## Key Files
- CLAUDE.md, DOMAINS.md, gsd-ultra.json (read from project root)
- Phase plans/summaries in: {phase_dir}/
```

## File Ownership Protocol

Ultra enforces domain-level file ownership during parallel execution.

### Ownership Levels

| Level | Permission | Example |
|-------|-----------|---------|
| **OWN** | Full read/write | `src/features/auth/` |
| **SHARED (append-only)** | Add new items, don't modify existing | `src/types/` |
| **READ-ONLY** | Import and reference, don't modify | `src/components/` |
| **DO NOT TOUCH** | No access | Other domain directories |

### Configuration

File ownership is defined in two places:
1. `DOMAINS.md` — human-readable domain decomposition
2. `gsd-ultra.json` — machine-readable domain paths and integration points

### Enforcement

File ownership is enforced at the **prompt level** — each executor agent's prompt includes ownership declarations. This is convention-based (not filesystem-level), relying on the agent following its instructions.

```
YOU OWN: src/features/auth/ (full read/write)
SHARED (append-only): src/types/ (add new types, don't modify existing)
DO NOT TOUCH: src/features/tasks/, src/features/dashboard/, src/features/notifications/
```

### File Ownership Spawn Template

Every executor/fixer spawn prompt MUST include this ownership block:

```markdown
<file_ownership>
**Domain:** {domain_name}

YOU OWN (full read/write):
- {owned_path_1}/
- {owned_path_2}/

SHARED (append-only — add new items, don't modify existing):
- {shared_path_1}
- {shared_path_2}

READ-ONLY (import and reference, don't modify):
- {readonly_path_1}/
- {readonly_path_2}/

DO NOT TOUCH (off-limits — other domain directories):
- {forbidden_path_1}/
- {forbidden_path_2}/

CRITICAL: If you need to modify a file outside your OWN or SHARED paths,
STOP and report it as a deviation. Do NOT modify it.

CROSS-DOMAIN DISCOVERY: If you discover something that affects another domain,
report it in your summary. The lead will handle cross-domain coordination.
</file_ownership>
```

## RELAY Protocol

**Cross-Researcher Discovery Notification**

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

**Agent Teams mode:** Researchers message the target teammate directly instead of writing RELAY sections in files. The mailbox IS the relay.

## Research Scaling Guidance

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

## RESEARCH.md Synthesis Template

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

### Synthesis Rules

**Job 1: Cross-Perspective Synthesis** — Combine findings into a coherent research document.

**Job 2: RELAY Integration** — Read RELAY sections from each researcher. Weave cross-discoveries into the relevant sections. Note which RELAYs led to new insights vs. were already covered.

**Job 3: Conflict Resolution** — When researchers disagree:
1. **Safety > convenience** — Risk Analyst's concerns override Domain Expert's recommendations
2. **Evidence > opinion** — the perspective with more concrete evidence wins
3. **Flag when uncertain** — if neither side has clear evidence, mark as unresolved for human review
4. **Document all conflicts** — even resolved ones, so the planner understands trade-offs

## VERIFICATION.md Template

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
{2-3 sentence summary of the verification outcome}

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

## Debate Results (Round 2)
| ATK-ID | Finding | Defender Response | Auditor Ruling | Final Severity |
|--------|---------|-------------------|----------------|----------------|

## Consensus Findings
{issues flagged by all 3 — strongest signals}

## Must-Have Status (Combined)
| Must-Have | Defender | Attacker | Auditor | Debate | Final |
|-----------|----------|----------|---------|--------|-------|

## Gaps (If Any)
{structured gaps in YAML frontmatter format for /ultra:gap-close}

## Human Verification Needed
{items requiring human testing}

## Recommendations
{what to do next based on verdict}
```

### Verdict Thresholds

**PASS** — All of:
- Defender: >=90% must-haves verified with STRONG evidence
- Attacker: 0 critical findings (post-debate), <=2 high findings (post-debate)
- Auditor: >=90% compliance score
- Consensus: 0 consensus findings unresolved

**CONDITIONAL_PASS** — All of:
- Defender: >=70% must-haves verified
- Attacker: 0 critical findings (post-debate)
- Auditor: >=70% compliance score
- No single issue flagged by all 3 roles remains unresolved

**FAIL** — Any of:
- Defender: <70% must-haves verified
- Attacker: >=1 critical finding (post-debate)
- Auditor: <70% compliance
- Consensus failure remains unresolved

## Bug Clustering Rules

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

## Agent Type Selection

| Bug Type | Agent | Rationale |
|----------|-------|-----------|
| Build errors / compile failures | gsd-executor | Needs to write/fix code, follows plan |
| Logic bugs / wrong behavior | gsd-debugger | Needs to investigate root cause |
| Missing features / stubs | gsd-executor | Needs to implement missing functionality |
| CSS / layout issues | gsd-executor | Needs to write UI code |
| Wiring / integration gaps | gsd-executor | Needs to connect components |
| Security vulnerabilities | gsd-executor | Needs to implement security patterns |
| Performance issues | gsd-debugger | Needs to profile and optimize |

## Ralph Self-Verify Protocol

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

### Ralph Results Format

```markdown
## Ralph Self-Verify Results
| Level | Status | Details |
|-------|--------|---------|
| Code Review | PASS/FAIL | {details} |
| Logical Walk-Through | PASS/FAIL | {details} |
| Runtime | PASS/FAIL | {details} |
```

### Fixer Spawn Template

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

**File Ownership:**
- OWN: {owned_paths}
- SHARED: {shared_paths}
- DO NOT TOUCH: {forbidden_paths}

**Ralph Self-Verify Protocol:**
After fixing, you MUST verify at 3 levels (see ultra-conventions.md).
Report Ralph results in your summary.
</fix_context>
```

## Hub-Spoke Communication

During parallel execution, all cross-domain discovery goes through the lead, not peer-to-peer.

Teammates report cross-domain discoveries in their summaries:

```markdown
## Cross-Domain Discoveries
- Domain B needs a `UserProfile` type that doesn't exist yet
- The API endpoint Domain A created returns paginated data, not a flat array
```

The lead processes these after the wave completes:
1. If it affects the next wave -> inject into next wave's prompts
2. If it requires backfilling -> create a mini integration task
3. If it's informational -> note in the integration summary

## Execution Scaling Guidance

| Phase Complexity | Teammates | Notes |
|-----------------|-----------|-------|
| Small (1-2 plans) | 1-2 | Standard GSD execution, no integration needed |
| Medium (3-4 plans) | 3-4 | Sweet spot, manageable integration |
| Large (5-6 plans) | 4-5 | Maximum recommended; interface contracts required |
| Very Large (7+) | Split phase | Break into sub-phases; 7+ teammates = coordination overhead exceeds gains |

**Coordination overhead curve:**
- 2 teammates: ~5% overhead (near-linear speedup)
- 4 teammates: ~15% overhead (still strong gains)
- 6 teammates: ~30% overhead (diminishing returns)
- 8+ teammates: >50% overhead (net negative, split the phase)

## Adversarial Planning Protocol

### Builder/Critic Debate Loop

```
Round 1: Builder creates -> Critic reviews
Round N: Builder revises -> Critic re-reviews
Exit: Critic signals APPROVED (0 BLOCKERs) OR max rounds reached
```

### Planning Auction (Advanced Mode)

For critical architecture phases, planning auction spawns 3 competing Builders who each create independent plans. The Critic evaluates all 3, selects the best, and the winning plan enters the standard Builder/Critic refinement loop. Use when:
- The phase involves a major architectural decision
- Multiple valid approaches exist and you want to explore them
- Quality matters more than speed

Not a separate command -- invoke via `/ultra:adversarial-plan {X} --auction`.

### Severity Levels

| Level | Meaning | Action Required |
|-------|---------|----------------|
| **BLOCKER** | Plan cannot proceed | Builder MUST fix |
| **WARNING** | Plan is weak here | Builder SHOULD fix |
| **SUGGESTION** | Plan could be better | Builder MAY fix |

### Convergence Detection

The Critic signals convergence by returning `## CRITIC REVIEW: APPROVED`.

Convergence = zero BLOCKERs remaining. Warnings and suggestions are documented but non-blocking.

### Max Rounds

Default: 5 rounds. Configurable in `gsd-ultra.json` under `pipeline.adversarial_plan.max_rounds`.

When max rounds reached without convergence:
1. Display remaining BLOCKERs
2. Offer: force-proceed, provide guidance, abort
3. User decides

## Adversarial Verification Protocol

### 3-Round Debate System

| Round | What Happens | Participants |
|-------|-------------|-------------|
| **Round 1** | Independent assessment (parallel) | Defender, Attacker, Auditor |
| **Round 2** | Structured debate | Attacker presents -> Defender responds -> Auditor rules |
| **Round 3** | Consensus (if needed) | Orchestrator resolves SPLIT rulings |

### Finding IDs and Debate Protocol

- Attacker uses ATK-{N} IDs for all findings
- Defender uses DEF-{N} IDs for all evidence
- Debate responses: **Concede** / **Dispute** (with evidence) / **Mitigate**
- Auditor rulings: **SUSTAINED** / **OVERRULED** / **SPLIT**

### Consensus Analysis

When all 3 roles flag the same issue -> **consensus failure** (very strong signal, auto-confirmed).
When roles disagree -> document discrepancy, debate in Round 2, Auditor rules.

## Knowledge Flywheel Protocol

The flywheel is Ultra's compound learning mechanism (see `references/knowledge-flywheel.md`).

### RETROSPECTIVE.md

Created by `/ultra:retrospective` after verification passes:
- What Delivered, What Went Well, What Went Wrong, What We Learned
- Process Metrics (plans, tasks, verdict, gap-close rounds)
- Patterns Identified (candidates for CLAUDE.md conventions)
- Architectural Decisions (candidates for STATE.md DEC-{NNN})
- Domain Boundary Changes (candidates for DOMAINS.md updates)

### STATE.md Decision Protocol

Decisions accumulate with DEC-{NNN} IDs:
```markdown
### DEC-001: {Title}
**Phase:** {X} -- {Name}
**Date:** {timestamp}
**Decision:** {what}
**Rationale:** {why}
```

### CLAUDE.md Evolution

Conventions are proposed during retrospective, approved by user:
1. Pattern identified -> Convention drafted -> User asked -> Approved/Modified/Rejected
2. Only specific, actionable conventions (not vague guidelines)

## Modes Reference

| Mode | Pipeline | Best For | Cost |
|------|----------|----------|------|
| **Standard GSD** | Research -> Plan -> Execute -> Verify | Most phases | Low |
| **Ultra Pipeline** | Swarm -> Debate Plan -> Parallel Execute -> Triple Verify -> Fix -> Learn | Complex/multi-domain | Medium |
| **Ultra Full** | `/ultra:full-pipeline` chains all 6 stages | Maximum quality | High |
| **Ultra Eco** | Ultra pipeline with eco model profile | Budget-conscious Ultra | Low |
| **Quick** | `/gsd:quick` -- plan + execute, no research/verify | Bug fixes, small tasks | Minimal |
| **Audit Only** | `/ultra:adversarial-verify` on existing code | Verify before release | Medium |
| **Research Only** | `/ultra:research-swarm` before GSD planning | Explore before committing | Low |

## Model Routing

Ultra uses `gsd-ultra.json` for model routing, falling back to GSD's `gsd-tools.js` model resolution.

### Resolution Priority

1. `gsd-ultra.json` -> `model_routing.roles.{role}[{profile}]`
2. GSD `gsd-tools.js resolve-model {agent-type}` (fallback)

### Profile Selection

Four profiles: `quality` / `balanced` / `budget` / `eco` set in `.planning/config.json`.

## Commit Conventions

Ultra commits follow GSD conventions with extended scope tags:

| Tag | Use |
|-----|-----|
| `docs(ultra):` | Pipeline artifacts (RESEARCH.md, PLAN.md, VERIFICATION.md, RETROSPECTIVE.md) |
| `feat(ultra-{domain}):` | Domain-scoped feature |
| `fix(ultra-{domain}):` | Domain-scoped fix |

## Research Swarm Output Structure

```
.planning/phases/XX-name/
+-- research/
|   +-- pattern-analysis.md      # From Pattern Analyst
|   +-- domain-expertise.md      # From Domain Expert
|   +-- risk-analysis.md         # From Risk Analyst
|   +-- ux-investigation.md      # From UX Investigator
+-- XX-RESEARCH.md               # Synthesized from all 4
+-- XX-01-PLAN.md                # From Builder (approved by Critic)
+-- XX-02-PLAN.md
+-- XX-01-SUMMARY.md             # From Executor
+-- XX-02-SUMMARY.md
+-- DEFENSE.md                   # From Defender
+-- ATTACK.md                    # From Attacker
+-- AUDIT.md                     # From Auditor
+-- DEBATE.md                    # From Round 2 (if applicable)
+-- XX-VERIFICATION.md           # Synthesized verdict
+-- RETROSPECTIVE.md             # From Knowledge Flywheel
```

## Dual-Mode Coordination (Agent Teams / Task Subagents)

Ultra supports two coordination modes. Every workflow detects which is available and branches accordingly. Both paths produce identical outputs -- same files, same format, same quality.

### Detection

Each workflow checks at startup:
```bash
AGENT_TEAMS_ENV=$(echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-0}")
AGENT_TEAMS_MODE=$( [ "$AGENT_TEAMS_ENV" = "1" ] && echo true || echo false )
```

### Enable Agent Teams

Set in `~/.claude/settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Or export before launching Claude Code:
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### Mode Comparison

| Feature | Task() Subagents (default) | Agent Teams (experimental) |
|---------|---------------------------|---------------------------|
| Communication | File relay (write -> read) | Native mailbox messaging |
| Task coordination | Orchestrator manages | Shared task list, self-claiming |
| Debate (verify) | Moderator Task() reads 3 files | Live messaging between teammates |
| RELAY (research) | Written sections in output files | Direct messages between researchers |
| Lead role | Orchestrates all calls | Delegate mode (coordinates only) |
| Completion signal | Task() return value | TeammateIdle / TaskCompleted hooks |
| Display | Sequential output | Split-pane (tmux/iTerm2) |
| Status | Stable | Experimental (Opus 4.6+, Feb 2026) |

### Mapping Table

| Ultra Pattern | Task() Implementation | Agent Teams Implementation |
|--------------|----------------------|---------------------------|
| RELAY protocol | `## RELAY -> Target` sections in files | Researcher messages Target teammate directly |
| Hub-spoke | Summaries -> lead reads -> injects into next prompts | Teammates message lead, lead messages affected teammates |
| Builder/Critic debate | Sequential Task() calls, orchestrator shuttles context | Direct Builder<->Critic messaging, lead monitors |
| Adversarial debate | Debate moderator Task() reads 3 files | Live Attacker<->Defender messaging, Auditor observes and rules |
| Parallel execution | Task() per domain with ownership blocks | Shared task list, teammates self-claim, delegate mode |
| Bug clustering | Task() per cluster | Shared task list from clusters, fixers self-claim |

### Fallback Guarantee

If Agent Teams is disabled or unavailable, all workflows fall back to existing Task() subagent behavior with zero changes to outputs. This is enforced by:
1. Detection step runs before any agent spawning
2. Each spawn step has explicit `if AGENT_TEAMS_MODE` / `else` branches
3. Output files and formats are identical regardless of mode

## Integration with GSD

Ultra commands coexist with GSD commands. You can mix them:

- Use `/gsd:new-project` to initialize (Ultra doesn't replace project init)
- Use `/ultra:research-swarm` instead of GSD's single researcher
- Use `/ultra:adversarial-plan` instead of GSD's plan-phase
- Use `/ultra:parallel-execute` instead of GSD's execute-phase
- Use `/ultra:adversarial-verify` instead of GSD's verify-work
- Use `/ultra:gap-close` to fix gaps from verification
- Use `/ultra:retrospective` to capture lessons learned
- Use `/ultra:full-pipeline` to chain all Ultra stages

GSD commands still work. Ultra is additive, not replacing.
