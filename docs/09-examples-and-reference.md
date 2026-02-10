# Examples and Reference

Worked examples and a quick reference cheat sheet for GSD Ultra.

## Example 1: Simple Feature (GSD Only)

**Scenario:** Add a dark mode toggle to an existing settings page.

```
/gsd:quick
> "Add dark mode toggle to settings page"
```

Single plan, single executor, atomic commit. No research or verification overhead needed.

**Cost:** ~$1-2, ~5 minutes.

## Example 2: Standard Phase (GSD Workflow)

**Scenario:** Phase 3 of a task management app — implement user authentication.

```
/gsd:discuss-phase 3     # "I want email/password login, no OAuth yet"
/gsd:plan-phase 3        # Researches auth patterns, creates 2 plans
/gsd:execute-phase 3     # Builds auth store, login form, API routes
/gsd:verify-work 3       # "Can you log in? Can you log out?"
```

Single researcher, single planner, wave-parallel execution, single verifier.

**Cost:** ~$5-8, ~20-30 minutes.

## Example 3: Complex Phase (Ultra Pipeline)

**Scenario:** Phase 5 of an e-commerce platform — multi-domain checkout flow touching auth, cart, payments, and notifications.

```
/ultra:research-swarm 5         # 4 researchers: patterns, domain, risk, UX
/ultra:adversarial-plan 5       # Builder creates plan, Critic debates 3 rounds
/ultra:parallel-execute 5       # 4 teammates execute domains in parallel
/ultra:adversarial-verify 5     # Defender, Attacker, Auditor triple-check
/ultra:gap-close 5              # Fix 3 findings, ralph self-verify each
/ultra:retrospective 5          # 2 conventions, 3 decisions logged
```

**What happened:**
- Research swarm found PCI compliance requirements (Risk Analyst) and Stripe best practices (Domain Expert)
- Builder/Critic debated for 3 rounds — Critic caught missing idempotency keys
- 4 domains executed in parallel: cart, payments, notifications, email
- Attacker found ATK-1: no rate limiting on payment endpoint, ATK-2: missing CSRF token
- Gap-close fixed both, Ralph verified at all 3 levels
- Retrospective added convention: "All POST endpoints must include CSRF token validation"

**Cost:** ~$13 (balanced profile), ~45 minutes.

## Example 4: Full Pipeline (Maximum Quality)

**Scenario:** Phase 1 of a new fintech application — critical architecture decisions needed.

```
/ultra:full-pipeline 1
```

Chains all 6 stages automatically. Uses planning auction for 3 competing architecture proposals.

**What happened:**
- 4 researchers discovered regulatory requirements, common fintech patterns, security risks, and UX standards
- 3 Builders proposed competing architectures (auction mode)
- Critic selected microservices approach, refined over 2 rounds
- 5 domains executed in parallel with strict file ownership
- Verification found 2 critical findings — both fixed in 1 gap-close round
- Retrospective logged 5 DEC-{NNN} decisions, 3 CLAUDE.md conventions

**Cost:** ~$27 (quality profile), ~90 minutes.

## Command Cheat Sheet

### GSD Core

| Command | When |
|---------|------|
| `/gsd:new-project` | Starting a new project |
| `/gsd:discuss-phase N` | Before planning — capture your vision |
| `/gsd:plan-phase N` | Create plans for a phase |
| `/gsd:execute-phase N` | Build a phase |
| `/gsd:verify-work N` | Test a phase works correctly |
| `/gsd:quick` | Bug fix or small task |

### Ultra Pipeline

| Command | When |
|---------|------|
| `/ultra:research-swarm N` | Complex phase needs multi-perspective research |
| `/ultra:adversarial-plan N` | Architecture decisions need debate |
| `/ultra:parallel-execute N` | Multi-domain phase needs parallel execution |
| `/ultra:adversarial-verify N` | Quality-critical code needs triple-check |
| `/ultra:gap-close N` | Fix verification findings |
| `/ultra:retrospective N` | Learn from completed phase |
| `/ultra:full-pipeline N` | Chain all stages |

### Navigation

| Command | When |
|---------|------|
| `/gsd:progress` | Where am I? |
| `/gsd:help` | What commands exist? |
| `/gsd:set-profile <p>` | Change model profile |
| `/gsd:pause-work` | Stopping mid-session |
| `/gsd:resume-work` | Continuing from pause |

### Advanced

| Command | When |
|---------|------|
| `/ultra:adversarial-plan N --auction` | 3 competing plans |
| `/gsd:map-codebase` | Existing code before new-project |
| `/gsd:debug` | Systematic debugging |
| `/gsd:audit-milestone` | Before completing milestone |

## Mode Selection Guide

| Your Situation | Use |
|---------------|-----|
| Fix a typo | Just ask Claude directly |
| Bug fix, small feature | `/gsd:quick` |
| Standard 3-8 file phase | GSD workflow (discuss → plan → execute → verify) |
| Multi-domain phase, 5+ files | Ultra pipeline |
| Critical architecture, must be right | `/ultra:full-pipeline` with quality profile |
| Well-established project, routine phase | Ultra with eco profile |
| Exploring before committing | `/ultra:research-swarm` alone |
| Verifying existing code | `/ultra:adversarial-verify` alone |

## Artifact Quick Reference

| File | Created By | Purpose |
|------|-----------|---------|
| `PROJECT.md` | `/gsd:new-project` | Project vision and scope |
| `REQUIREMENTS.md` | `/gsd:new-project` | Scoped v1/v2 requirements |
| `ROADMAP.md` | `/gsd:new-project` | Phased delivery plan |
| `STATE.md` | `/gsd:new-project` + retrospective | Decisions, blockers, position |
| `CONTEXT.md` | `/gsd:discuss-phase` | Your implementation preferences |
| `RESEARCH.md` | Research agents | Domain knowledge |
| `PLAN.md` | Planning agents | Atomic task plans |
| `SUMMARY.md` | Executors | What was built |
| `DEFENSE.md` | Defender | Evidence matrix |
| `ATTACK.md` | Attacker | Bug findings |
| `AUDIT.md` | Auditor | Compliance score |
| `DEBATE.md` | Round 2 debate | Structured arguments |
| `VERIFICATION.md` | Verification synthesis | Verdict + gaps |
| `RETROSPECTIVE.md` | `/ultra:retrospective` | Lessons learned |
| `CLAUDE.md` | Retrospective + user | Project conventions |
| `DOMAINS.md` | Ultra setup | Domain ownership map |
| `gsd-ultra.json` | Manual | Ultra configuration |
| `.planning/config.json` | Auto-created | GSD settings |
