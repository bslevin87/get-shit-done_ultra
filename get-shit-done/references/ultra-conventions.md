# Ultra Conventions Reference

Ultra extends GSD with multi-perspective research, adversarial planning, and adversarial verification. These conventions define how Ultra components interact.

## Pipeline Stages

| Stage | Command | Agents | Output |
|-------|---------|--------|--------|
| 1. Research Swarm | `/ultra:research-swarm` | Pattern Analyst, Domain Expert, Risk Analyst, UX Investigator | 4 perspective files + RESEARCH.md |
| 2. Adversarial Plan | `/ultra:adversarial-plan` | Builder, Critic | PLAN.md files (approved) |
| 3. Parallel Execute | `/ultra:parallel-execute` | GSD Executors (with file ownership) | SUMMARY.md files |
| 4. Adversarial Verify | `/ultra:adversarial-verify` | Defender, Attacker, Auditor | DEFENSE/ATTACK/AUDIT.md + VERIFICATION.md |
| 5. Gap Close | `/ultra:gap-close` | GSD Executors | Gap fix commits |

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

### Triple-Check System

| Role | Perspective | Output |
|------|-------------|--------|
| **Defender** | Advocate — what works | DEFENSE.md |
| **Attacker** | Adversary — what's broken | ATTACK.md |
| **Auditor** | Neutral — compliance | AUDIT.md |

### Verdict Logic

| Verdict | Conditions |
|---------|-----------|
| **PASS** | Defender ≥90%, Attacker 0 critical + ≤2 high, Auditor ≥90% |
| **CONDITIONAL_PASS** | Defender ≥70%, Attacker 0 critical, Auditor ≥70% |
| **FAIL** | Any role below thresholds, OR critical finding |

### Consensus Analysis

When all 3 roles flag the same issue → **consensus failure** (very strong signal).
When roles disagree → document discrepancy and weigh evidence.

## Model Routing

Ultra uses `gsd-ultra.json` for model routing, falling back to GSD's `gsd-tools.js` model resolution.

### Resolution Priority

1. `gsd-ultra.json` → `model_routing.roles.{role}[{profile}]`
2. GSD `gsd-tools.js resolve-model {agent-type}` (fallback)

### Profile Selection

Same as GSD: `quality` / `balanced` / `budget` set in `.planning/config.json`.

## Commit Conventions

Ultra commits follow GSD conventions with extended scope tags:

| Tag | Use |
|-----|-----|
| `docs(ultra):` | Pipeline artifacts (RESEARCH.md, PLAN.md, VERIFICATION.md) |
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
└── XX-VERIFICATION.md           # Synthesized verdict
```

## Integration with GSD

Ultra commands coexist with GSD commands. You can mix them:

- Use `/gsd:new-project` to initialize (Ultra doesn't replace project init)
- Use `/ultra:research-swarm` instead of GSD's single researcher
- Use `/ultra:adversarial-plan` instead of GSD's plan-phase
- Use `/ultra:parallel-execute` instead of GSD's execute-phase
- Use `/ultra:adversarial-verify` instead of GSD's verify-work
- Use `/ultra:full-pipeline` to chain all Ultra stages

GSD commands still work. Ultra is additive, not replacing.
