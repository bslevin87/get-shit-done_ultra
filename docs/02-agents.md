# Agents

GSD Ultra uses 20 specialized agents organized into families. Each agent has a focused role and runs in its own fresh context window.

## Agent Families

### Architect Family

| Agent | What it does |
|-------|-------------|
| `gsd-planner` | Creates atomic task plans with XML structure. Reads RESEARCH.md, CONTEXT.md, and STATE.md decisions. Outputs PLAN.md files with tasks, file lists, verification steps. |
| `gsd-roadmapper` | Creates phased roadmaps from requirements. Maps each requirement to a phase, ensures coverage, produces ROADMAP.md. |
| `ultra-builder` | Creates plans with interface contracts and domain-to-wave assignment. File ownership declarations in every plan. Reads STATE.md DEC-{NNN} decisions. |

### Builder Family

| Agent | What it does |
|-------|-------------|
| `gsd-executor` | Implements plans in fresh context windows. Follows XML task structure, commits per task, writes SUMMARY.md. |
| `gsd-debugger` | Investigates bugs using scientific method. Manages debug sessions with checkpoints. Persistent state across context resets. |

### Research Family

| Agent | What it does |
|-------|-------------|
| `gsd-phase-researcher` | Investigates how to implement a phase. Produces RESEARCH.md consumed by the planner. |
| `gsd-project-researcher` | Investigates domain ecosystem for new projects. Spawned during `/gsd:new-project`. |
| `gsd-research-synthesizer` | Synthesizes parallel researcher outputs into unified RESEARCH.md. Resolves conflicts. |
| `gsd-codebase-mapper` | Explores existing codebases with parallel agents. Produces structured analysis documents. |
| `ultra-pattern-analyst` | Research swarm — finds patterns in the ecosystem, existing implementations, library options. |
| `ultra-domain-expert` | Research swarm — deep domain knowledge, best practices, common pitfalls. |
| `ultra-risk-analyst` | Research swarm — identifies risks, trade-offs, security concerns, performance implications. |
| `ultra-ux-investigator` | Research swarm — UX patterns, accessibility, responsive design, interaction models. |

### Quality Family

| Agent | What it does |
|-------|-------------|
| `gsd-verifier` | Checks codebase delivers what phase promised. Goal-backward verification. |
| `gsd-plan-checker` | Verifies plans achieve phase goals before execution starts. |
| `gsd-integration-checker` | Verifies cross-phase integration and end-to-end flows. |
| `ultra-critic` | Attacks plans with BLOCKER / WARNING / SUGGESTION findings. Loops with Builder until convergence. |
| `ultra-defender` | Builds evidence matrix mapping requirements to file:line proof. Uses DEF-{N} IDs. Responds to attacks with Concede or Dispute (with evidence). |
| `ultra-attacker` | Hunts bugs with composite expertise: critic + security auditor + edge-case hunter. Uses ATK-{N} finding IDs. |
| `ultra-auditor` | Weighted compliance score across 6 dimensions. Rules on disputes: SUSTAINED / OVERRULED / SPLIT. |

## Composite Behavior

Ultra verification agents embody multiple expertise domains:

### Attacker Composite

The Attacker doesn't just find bugs — it thinks like three specialists simultaneously:

- **Critic** — Logic errors, missing edge cases, incomplete implementations
- **Security Auditor** — XSS vectors, injection points, authentication gaps
- **Edge-Case Hunter** — Empty states, null handling, boundary conditions

Each finding is tagged with the composite perspective that discovered it.

### Auditor Composite

The Auditor evaluates code from three angles:

- **Code Reviewer** — Quality, patterns, maintainability
- **Standards Checker** — Convention compliance, consistency
- **Accessibility Auditor** — ARIA labels, keyboard navigation, screen readers

The compliance score is a weighted average:
- Plan Adherence: 30%
- Convention Compliance: 20%
- Requirements Coverage: 20%
- File Ownership: 15%
- Commit Standards: 10%
- Accessibility: 5%

### Defender Composite

The Defender acts as advocate and evidence collector:

- **Advocate** — Presents the strongest case for the implementation
- **Evidence Collector** — Maps every requirement to file:line proof

Evidence matrix format:
```
| DEF-1 | User can log in | LoginForm handles submit → authStore.login() | src/features/auth/LoginForm.tsx:42 | STRONG |
```

## Model Assignment

Agents use different Claude models based on the active profile:

| Agent | quality | balanced | budget | eco |
|-------|---------|----------|--------|-----|
| gsd-planner | opus | opus | sonnet | sonnet |
| gsd-executor | opus | sonnet | sonnet | sonnet |
| ultra-builder | opus | opus | sonnet | sonnet |
| ultra-attacker | opus | sonnet | sonnet | haiku |
| ultra-auditor | sonnet | sonnet | haiku | haiku |

See [Configuration and Cost](08-configuration-and-cost.md) for the full model profile table.

## Agent Tiers

**Tier 1: Architects** — Make design decisions. Need highest reasoning capability. Use Opus in quality/balanced profiles.

**Tier 2: Builders** — Follow explicit plans. Plans contain the reasoning; execution is implementation. Sonnet is sufficient in balanced mode.

**Tier 3: Researchers** — Read-only exploration. Structured output from file contents. Can use Haiku in budget/eco profiles.

**Tier 4: Quality** — Goal-backward reasoning. Need to assess whether code delivers what was promised, not just pattern match. Sonnet minimum recommended.
