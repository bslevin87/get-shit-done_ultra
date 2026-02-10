# Pipeline Overview

Ultra extends GSD with a 6-stage pipeline. Each stage is independently invocable — use what you need.

## The 6 Stages

```
┌─────────────┐    ┌──────────────┐    ┌──────────────┐
│  1. RESEARCH │───▶│  2. PLAN     │───▶│  3. EXECUTE  │
│  Swarm ×4    │    │  Builder vs  │    │  Parallel    │
│  perspectives│    │  Critic      │    │  domains     │
└─────────────┘    └──────────────┘    └──────────────┘
       │                                       │
       ▼                                       ▼
┌─────────────┐    ┌──────────────┐    ┌──────────────┐
│  6. LEARN   │◀───│  5. FIX      │◀───│  4. VERIFY   │
│  Retrospect │    │  Cluster +   │    │  Defender vs  │
│  + flywheel │    │  ralph-verify│    │  Attacker vs  │
└─────────────┘    └──────────────┘    │  Auditor     │
                                       └──────────────┘
```

## Coordination Layer

All Ultra workflows support dual-mode coordination:

- **Task() Subagents (default):** File-based relay, orchestrator manages all calls. Stable, works everywhere.
- **Agent Teams (experimental):** Native teammate messaging, shared task lists, delegate mode. Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.

Each workflow detects the mode at startup and branches accordingly. Both paths produce identical output files. See [Configuration](08-configuration-and-cost.md) for setup.

## Stage Details

### Stage 1: Research Swarm

**Command:** `/ultra:research-swarm <N>`
**Agents:** Pattern Analyst, Domain Expert, Risk Analyst, UX Investigator
**Output:** 4 perspective files + synthesized `RESEARCH.md`

Four researchers investigate simultaneously. Each brings a different lens — patterns in the ecosystem, deep domain knowledge, risk assessment, and UX considerations. The RELAY protocol lets researchers flag discoveries for each other. A synthesizer resolves conflicts and produces a unified research document.

### Stage 2: Adversarial Plan

**Command:** `/ultra:adversarial-plan <N>`
**Agents:** Builder, Critic
**Output:** Approved `PLAN.md` files

The Builder creates plans. The Critic attacks them with BLOCKER / WARNING / SUGGESTION findings. The Builder revises. Loop until zero BLOCKERs or max 5 rounds. Plans that survive this scrutiny rarely fail during execution.

**Advanced:** Add `--auction` to spawn 3 competing Builders. The Critic evaluates all 3, selects the best, then enters the standard refinement loop.

### Stage 3: Parallel Execute

**Command:** `/ultra:parallel-execute <N>`
**Agents:** GSD Executors (domain-scoped)
**Output:** `SUMMARY.md` files per plan

Domains execute in parallel with strict file ownership. Each executor owns specific directories (OWN), can append to shared files (SHARED), and must not touch other domains (DO NOT TOUCH). After all teammates complete, the lead performs deferred integration — wiring domains together, checking for conflicts, running build verification.

### Stage 4: Adversarial Verify

**Command:** `/ultra:adversarial-verify <N>`
**Agents:** Defender, Attacker, Auditor
**Output:** `DEFENSE.md`, `ATTACK.md`, `AUDIT.md`, `DEBATE.md`, `VERIFICATION.md`

Three independent assessments in parallel (Round 1), then structured debate (Round 2), then consensus resolution if needed (Round 3). The Attacker uses ATK-N finding IDs; the Defender responds with Concede, Dispute (with evidence), or Mitigate; the Auditor rules SUSTAINED, OVERRULED, or SPLIT.

**Verdict:** PASS (90%+), CONDITIONAL_PASS (70%+), or FAIL.

### Stage 5: Gap Close

**Command:** `/ultra:gap-close <N>`
**Agents:** GSD Executors and/or Debuggers
**Output:** Fix commits, updated `VERIFICATION.md`

Clusters related bugs (same file, same domain, cross-domain). Selects the right agent type — executors for missing features, debuggers for logic bugs. Each fixer runs the Ralph 3-level self-verify: code review, logical walk-through, runtime verification. Up to 5 retries with escalation analysis.

### Stage 6: Retrospective

**Command:** `/ultra:retrospective <N>`
**Agents:** Orchestrator (interactive)
**Output:** `RETROSPECTIVE.md`, CLAUDE.md updates, STATE.md decisions

The knowledge flywheel. Analyzes completed work, extracts patterns, proposes CLAUDE.md conventions (user approves each), logs DEC-{NNN} architectural decisions to STATE.md, reviews domain boundaries. Every phase makes the next one faster.

## Full Pipeline

```
/ultra:full-pipeline <N>
```

Chains all 6 stages. Maximum quality, zero manual orchestration.

## Mixing GSD and Ultra

Ultra is additive. You can mix commands freely:

- Use `/gsd:new-project` to initialize (Ultra doesn't replace project init)
- Use `/ultra:research-swarm` instead of GSD's single researcher
- Use GSD's `/gsd:plan-phase` for simple phases, Ultra's `/ultra:adversarial-plan` for complex ones
- Use `/gsd:execute-phase` or `/ultra:parallel-execute` depending on domain count
- Use `/ultra:adversarial-verify` for critical phases, `/gsd:verify-work` for simple ones

## Artifacts

Each stage produces artifacts in `.planning/phases/{XX}-{name}/`:

```
.planning/phases/03-auth/
├── research/
│   ├── pattern-analysis.md
│   ├── domain-expertise.md
│   ├── risk-analysis.md
│   └── ux-investigation.md
├── 03-RESEARCH.md
├── 03-01-PLAN.md
├── 03-02-PLAN.md
├── 03-01-SUMMARY.md
├── 03-02-SUMMARY.md
├── DEFENSE.md
├── ATTACK.md
├── AUDIT.md
├── DEBATE.md
├── 03-VERIFICATION.md
└── RETROSPECTIVE.md
```
