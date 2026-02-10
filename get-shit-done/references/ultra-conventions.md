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

## Adversarial Planning Protocol

### Builder/Critic Debate Loop

```
Round 1: Builder creates → Critic reviews
Round N: Builder revises → Critic re-reviews
Exit: Critic signals APPROVED (0 BLOCKERs) OR max rounds reached
```

### Planning Auction (Advanced Mode)

For critical architecture phases, planning auction spawns 3 competing Builders who each create independent plans. The Critic evaluates all 3, selects the best, and the winning plan enters the standard Builder/Critic refinement loop. Use when:
- The phase involves a major architectural decision
- Multiple valid approaches exist and you want to explore them
- Quality matters more than speed

Not a separate command — invoke via `/ultra:adversarial-plan {X} --auction`.

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
| **Round 2** | Structured debate | Attacker presents → Defender responds → Auditor rules |
| **Round 3** | Consensus (if needed) | Orchestrator resolves SPLIT rulings |

### Finding IDs and Debate Protocol

- Attacker uses ATK-{N} IDs for all findings
- Defender uses DEF-{N} IDs for all evidence
- Debate responses: **Concede** / **Dispute** (with evidence) / **Mitigate**
- Auditor rulings: **SUSTAINED** / **OVERRULED** / **SPLIT**

### Verdict Logic

| Verdict | Conditions |
|---------|-----------|
| **PASS** | Defender ≥90%, Attacker 0 critical + ≤2 high (post-debate), Auditor ≥90% |
| **CONDITIONAL_PASS** | Defender ≥70%, Attacker 0 critical (post-debate), Auditor ≥70% |
| **FAIL** | Any role below thresholds, OR critical finding, OR unresolved consensus failure |

### Consensus Analysis

When all 3 roles flag the same issue → **consensus failure** (very strong signal, auto-confirmed).
When roles disagree → document discrepancy, debate in Round 2, Auditor rules.

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
**Phase:** {X} — {Name}
**Date:** {timestamp}
**Decision:** {what}
**Rationale:** {why}
```

### CLAUDE.md Evolution

Conventions are proposed during retrospective, approved by user:
1. Pattern identified → Convention drafted → User asked → Approved/Modified/Rejected
2. Only specific, actionable conventions (not vague guidelines)

## Modes Reference

| Mode | Pipeline | Best For | Cost |
|------|----------|----------|------|
| **Standard GSD** | Research → Plan → Execute → Verify | Most phases | Low |
| **Ultra Pipeline** | Swarm → Debate Plan → Parallel Execute → Triple Verify → Fix → Learn | Complex/multi-domain | Medium |
| **Ultra Full** | `/ultra:full-pipeline` chains all 6 stages | Maximum quality | High |
| **Ultra Eco** | Ultra pipeline with eco model profile | Budget-conscious Ultra | Low |
| **Quick** | `/gsd:quick` — plan + execute, no research/verify | Bug fixes, small tasks | Minimal |
| **Audit Only** | `/ultra:adversarial-verify` on existing code | Verify before release | Medium |
| **Research Only** | `/ultra:research-swarm` before GSD planning | Explore before committing | Low |

## Model Routing

Ultra uses `gsd-ultra.json` for model routing, falling back to GSD's `gsd-tools.js` model resolution.

### Resolution Priority

1. `gsd-ultra.json` → `model_routing.roles.{role}[{profile}]`
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
├── research/
│   ├── pattern-analysis.md      # From Pattern Analyst
│   ├── domain-expertise.md      # From Domain Expert
│   ├── risk-analysis.md         # From Risk Analyst
│   └── ux-investigation.md      # From UX Investigator
├── XX-RESEARCH.md               # Synthesized from all 4
├── XX-01-PLAN.md                # From Builder (approved by Critic)
├── XX-02-PLAN.md
├── XX-01-SUMMARY.md             # From Executor
├── XX-02-SUMMARY.md
├── DEFENSE.md                   # From Defender
├── ATTACK.md                    # From Attacker
├── AUDIT.md                     # From Auditor
├── DEBATE.md                    # From Round 2 (if applicable)
├── XX-VERIFICATION.md           # Synthesized verdict
└── RETROSPECTIVE.md             # From Knowledge Flywheel
```

## Dual-Mode Coordination (Agent Teams / Task Subagents)

Ultra supports two coordination modes. Every workflow detects which is available and branches accordingly. Both paths produce identical outputs — same files, same format, same quality.

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
| Communication | File relay (write → read) | Native mailbox messaging |
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
| RELAY protocol | `## RELAY → Target` sections in files | Researcher messages Target teammate directly |
| Hub-spoke | Summaries → lead reads → injects into next prompts | Teammates message lead, lead messages affected teammates |
| Builder/Critic debate | Sequential Task() calls, orchestrator shuttles context | Direct Builder↔Critic messaging, lead monitors |
| Adversarial debate | Debate moderator Task() reads 3 files | Live Attacker↔Defender messaging, Auditor observes and rules |
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
