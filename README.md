<div align="center">

# GET SHIT DONE · ULTRA

**GSD makes Claude Code reliable. Ultra makes it extraordinary.**

**3x quality. 2.5x speed. Same no-BS philosophy.**

[![Discord](https://img.shields.io/badge/Discord-Join-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/5JJgD5svVS)
[![GitHub stars](https://img.shields.io/github/stars/bslevin87/get-shit-done_ultra?style=for-the-badge&logo=github&color=181717)](https://github.com/bslevin87/get-shit-done_ultra)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)

<br>

```bash
npx get-shit-done-cc@latest
```

**Works on Mac, Windows, and Linux.**

<br>

![GSD Install](assets/terminal.svg)

<br>

*"If you know clearly what you want, this WILL build it for you. No bs."*

*"I've done SpecKit, OpenSpec and Taskmaster — this has produced the best results for me."*

*"By far the most powerful addition to my Claude Code. Nothing over-engineered. Literally just gets shit done."*

<br>

**Trusted by engineers at Amazon, Google, Shopify, and Webflow.**

[Why Ultra](#why-ultra) · [How It Works](#how-it-works) · [The Ultra Pipeline](#the-ultra-pipeline) · [Commands](#commands) · [Knowledge Flywheel](#the-knowledge-flywheel) · [When to Use What](#when-to-use-what)

</div>

---

## Why Ultra

GSD is a solid foundation — context rot solved, fresh context per plan, atomic commits, structured plans. It works. Engineers at real companies use it every day.

But after running it on complex, multi-domain builds, I kept hitting the same four walls:

1. **Blind spots** — A single researcher misses what a team of four would catch. One perspective isn't enough for complex phases.
2. **Weak plans** — Plans look good until you're three tasks in and realize the architecture doesn't hold. No one challenged the assumptions.
3. **Bugs slip through** — A single verifier does its best, but it can't simultaneously be the defender, the attacker, and the impartial judge.
4. **No learning** — Phase 8 is as expensive as Phase 1. The system doesn't get smarter from what it's already built.

Ultra is my fix for all four:

| Problem | GSD | Ultra |
|---------|-----|-------|
| Blind spots | 1 researcher | 4-perspective research swarm |
| Weak plans | Single planner | Builder/Critic adversarial debate |
| Bugs slip | Single verifier | Defender + Attacker + Auditor triple-check |
| No learning | Fresh start every phase | Knowledge flywheel — conventions compound |

Ultra is GSD's natural evolution, not a bolt-on. Every Ultra command enriches what's already there. You can use GSD commands, Ultra commands, or mix them freely.

---

## Getting Started

```bash
npx get-shit-done-cc@latest
```

The installer prompts you to choose:
1. **Runtime** — Claude Code, OpenCode, Gemini, or all
2. **Location** — Global (all projects) or local (current project only)

Verify with `/gsd:help` inside your chosen runtime.

### Staying Updated

GSD evolves fast. Update periodically:

```bash
npx get-shit-done-cc@latest
```

<details>
<summary><strong>Non-interactive Install (Docker, CI, Scripts)</strong></summary>

```bash
# Claude Code
npx get-shit-done-cc --claude --global   # Install to ~/.claude/
npx get-shit-done-cc --claude --local    # Install to ./.claude/

# OpenCode (open source, free models)
npx get-shit-done-cc --opencode --global # Install to ~/.config/opencode/

# Gemini CLI
npx get-shit-done-cc --gemini --global   # Install to ~/.gemini/

# All runtimes
npx get-shit-done-cc --all --global      # Install to all directories
```

Use `--global` (`-g`) or `--local` (`-l`) to skip the location prompt.
Use `--claude`, `--opencode`, `--gemini`, or `--all` to skip the runtime prompt.

</details>

<details>
<summary><strong>Development Installation</strong></summary>

Clone the repository and run the installer locally:

```bash
git clone https://github.com/bslevin87/get-shit-done_ultra.git
cd get-shit-done_ultra
node bin/install.js --claude --local
```

Installs to `./.claude/` for testing modifications before contributing.

</details>

### Agent Teams Mode (Experimental)

Ultra can use Claude Code's native Agent Teams for inter-agent coordination — live messaging instead of file relay, shared task lists instead of manual orchestration. This is experimental (Opus 4.6+, Feb 2026).

**Enable:**
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

Or add to `~/.claude/settings.json`:
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

When enabled, all Ultra workflows use native teammate coordination. When disabled (default), existing Task() subagent behavior is unchanged. Both modes produce identical output files.

| Feature | Task() Subagents | Agent Teams |
|---------|-----------------|-------------|
| Communication | File relay | Native mailbox |
| Debate (verify) | Moderator reads 3 files | Live messaging |
| RELAY (research) | Written sections in files | Direct messages |
| Task management | Orchestrator manages | Shared task list |
| Display | Sequential | Split-pane |

### Ultra Setup (Optional)

Ultra works out of the box. For multi-domain phases, add a `gsd-ultra.json` to your project root:

```json
{
  "domains": {
    "auth": { "paths": ["src/features/auth/"], "owner": "teammate-1" },
    "tasks": { "paths": ["src/features/tasks/"], "owner": "teammate-2" }
  },
  "shared_paths": ["src/types/", "src/lib/"],
  "pipeline": {
    "adversarial_plan": { "max_rounds": 5 },
    "adversarial_verify": { "max_debate_rounds": 3 }
  }
}
```

Not required. Without it, Ultra uses smart defaults and auto-discovers domains from your codebase.

### Recommended: Skip Permissions Mode

GSD is designed for frictionless automation. Run Claude Code with:

```bash
claude --dangerously-skip-permissions
```

> [!TIP]
> This is how GSD is intended to be used — stopping to approve `date` and `git commit` 50 times defeats the purpose.

<details>
<summary><strong>Alternative: Granular Permissions</strong></summary>

If you prefer not to use that flag, add this to your project's `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(echo:*)",
      "Bash(cat:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(wc:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(sort:*)",
      "Bash(grep:*)",
      "Bash(tr:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git tag:*)"
    ]
  }
}
```

</details>

---

## How It Works

The complete GSD Ultra flow — from idea to shipped code with compound learning:

```
                          ┌── GSD FOUNDATION ──┐    ┌─────────── ULTRA PIPELINE ───────────┐
                          │                     │    │                                       │
/gsd:new-project ──▶ /gsd:discuss-phase ──▶ RESEARCH ──▶ PLAN ──▶ EXECUTE ──▶ VERIFY ──▶ FIX ──▶ LEARN
    or                  (your vision)       Swarm ×4   Builder   Parallel   Defender    Cluster  Flywheel
/gsd:map-codebase                                      vs        domains    vs          + Ralph
                                                       Critic               Attacker
                                                                            vs Auditor
```

### Step 0: Initialize Your Project

> **New project?** → `/gsd:new-project` — In-depth interview about your idea. Creates `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`.
>
> **Existing code?** → `/gsd:map-codebase` first — Spawns parallel agents to analyze your stack, architecture, conventions, and concerns. Then `/gsd:new-project` knows your codebase.

---

### Step 1: Discuss Phase — Shape Your Vision

```
/gsd:discuss-phase 1
```

**This is where you shape the implementation.** Your roadmap has a sentence or two per phase. That's not enough context to build something the way *you* imagine it.

The system analyzes the phase and identifies gray areas based on what's being built:

- **Visual features** → Layout, density, interactions, empty states
- **APIs/CLIs** → Response format, flags, error handling, verbosity
- **Content systems** → Structure, tone, depth, flow
- **Organization tasks** → Grouping criteria, naming, duplicates, exceptions

For each area, it asks until you're satisfied. The output — `CONTEXT.md` — feeds directly into every downstream stage. The deeper you go here, the more the system builds what you actually want. Skip it and you get reasonable defaults. Use it and you get *your* vision.

---

### Step 2: Research Swarm — 4 Perspectives

```
/ultra:research-swarm 1
```

Four researchers investigate simultaneously, each bringing a different lens:

| Researcher | Answers |
|-----------|---------|
| **Pattern Analyst** | How have others solved this? Libraries, patterns, anti-patterns |
| **Domain Expert** | What does this domain require? Best practices, compliance |
| **Risk Analyst** | What could go wrong? Security, performance, dependencies |
| **UX Investigator** | How should users experience this? Accessibility, interactions |

Cross-perspective **RELAY protocol** — when a researcher discovers info affecting another's domain, they flag it. Synthesizer resolves conflicts (safety > convenience, evidence > opinion).

---

### Step 3: Adversarial Plan — Builder vs Critic

```
/ultra:adversarial-plan 1
```

Builder creates the plan. Critic attacks it with BLOCKER / WARNING / SUGGESTION findings. Builder revises. Critic re-reviews. Loop until zero BLOCKERs or max 5 rounds.

Plans include interface contracts between domains, file ownership declarations, and domain-to-wave assignment. Result: plans that survive scrutiny before a single line of code is written.

---

### Step 4: Parallel Execute — Domain Ownership

```
/ultra:parallel-execute 1
```

Domains execute in parallel with strict file ownership:

| Level | Permission |
|-------|-----------|
| **OWN** | Full read/write |
| **SHARED** | Append only — add new, don't modify existing |
| **READ-ONLY** | Import and reference only |
| **DO NOT TOUCH** | No access |

**Deferred integration** — teammates build domains independently, lead wires them together after all complete. No merge conflicts, no race conditions.

---

### Step 5: Adversarial Verify — Triple-Check Debate

```
/ultra:adversarial-verify 1
```

Three independent assessors, then structured debate:

- **Round 1:** Defender, Attacker, and Auditor assess independently (parallel)
- **Round 2:** Attacker presents ATK-N findings → Defender responds (Concede/Dispute/Mitigate) → Auditor rules (SUSTAINED/OVERRULED/SPLIT)
- **Round 3:** Consensus resolution for any SPLIT rulings

When all three flag the same issue = **consensus failure** (strongest signal). Verdict: **PASS** (90%+) / **CONDITIONAL_PASS** (70%+) / **FAIL**.

---

### Step 6: Gap Close — Cluster + Fix + Self-Verify

```
/ultra:gap-close 1
```

Clusters related bugs by file and domain — fixes root causes, not symptoms. Selects the right agent (executor for missing features, debugger for logic bugs). Each fixer runs **Ralph 3-level self-verify**: code review → logical walk-through → runtime verification. Up to 5 retries with escalation analysis.

---

### Step 7: Retrospective — The Flywheel

```
/ultra:retrospective 1
```

The compound learning mechanism. Extracts conventions from completed code, proposes CLAUDE.md additions (you approve each one), logs architectural decisions as DEC-{NNN} to STATE.md. Every phase makes the next one faster.

---

### Then Repeat

```
/gsd:discuss-phase 2
/ultra:research-swarm 2
/ultra:adversarial-plan 2
/ultra:parallel-execute 2
/ultra:adversarial-verify 2
/ultra:gap-close 2
/ultra:retrospective 2
...
/gsd:complete-milestone
```

Or chain everything for a phase:

```
/ultra:full-pipeline 2
```

### Quick Mode

```
/gsd:quick
```

For ad-hoc tasks that don't need the full pipeline. Same agents, same quality — skips research, debate, and verification. Use for bug fixes, small features, config changes.

---

## The Agent Roster

### GSD Agents (11)

| Agent | Role | Tier |
|-------|------|------|
| `gsd-planner` | Creates atomic task plans with XML structure | Architect |
| `gsd-executor` | Implements plans in fresh context windows | Builder |
| `gsd-debugger` | Investigates bugs with scientific method | Specialist |
| `gsd-verifier` | Checks codebase delivers what phase promised | Quality |
| `gsd-plan-checker` | Verifies plans achieve goals before execution | Quality |
| `gsd-integration-checker` | Verifies cross-phase E2E flows | Quality |
| `gsd-roadmapper` | Creates phased roadmaps from requirements | Architect |
| `gsd-phase-researcher` | Investigates how to implement a phase | Research |
| `gsd-project-researcher` | Investigates domain ecosystem for new projects | Research |
| `gsd-research-synthesizer` | Synthesizes parallel researcher outputs | Research |
| `gsd-codebase-mapper` | Explores and documents existing codebases | Research |

### Ultra Agents (9)

| Agent | Role | Composite Behavior |
|-------|------|--------------------|
| `ultra-pattern-analyst` | Research — finds patterns in ecosystem | — |
| `ultra-domain-expert` | Research — deep domain knowledge | — |
| `ultra-risk-analyst` | Research — identifies risks and trade-offs | — |
| `ultra-ux-investigator` | Research — UX patterns and accessibility | — |
| `ultra-builder` | Planning — creates plans with interface contracts | — |
| `ultra-critic` | Planning — attacks plans until zero BLOCKERs | — |
| `ultra-defender` | Verification — evidence matrix with file:line refs | Advocate + Evidence Collector |
| `ultra-attacker` | Verification — finds bugs with ATK-N IDs | Critic + Security Auditor + Edge-Case Hunter |
| `ultra-auditor` | Verification — weighted compliance score | Code Reviewer + Standards Checker + Accessibility Auditor |

---

## Commands

### Core GSD Workflow

| Command | What it does |
|---------|--------------|
| `/gsd:new-project [--auto]` | Full initialization: questions → research → requirements → roadmap |
| `/gsd:discuss-phase [N]` | Capture implementation decisions before planning |
| `/gsd:plan-phase [N]` | Research + plan + verify for a phase |
| `/gsd:execute-phase <N>` | Execute all plans in parallel waves, verify when complete |
| `/gsd:verify-work [N]` | Manual user acceptance testing |
| `/gsd:audit-milestone` | Verify milestone achieved its definition of done |
| `/gsd:complete-milestone` | Archive milestone, tag release |
| `/gsd:new-milestone [name]` | Start next version |

### Ultra Pipeline

| Command | What it does |
|---------|--------------|
| `/ultra:research-swarm <N>` | 4-perspective research swarm with RELAY + conflict resolution |
| `/ultra:adversarial-plan <N>` | Builder/Critic debate until APPROVED (max 5 rounds) |
| `/ultra:parallel-execute <N>` | Domain-parallel execution with file ownership |
| `/ultra:adversarial-verify <N>` | 3-round debate: Defender vs Attacker vs Auditor |
| `/ultra:gap-close <N>` | Bug clustering + ralph self-verify (max 5 retries) |
| `/ultra:retrospective <N>` | Knowledge flywheel: conventions + decisions + domain review |
| `/ultra:full-pipeline <N>` | Chain all 6 Ultra stages for a phase |

### Navigation

| Command | What it does |
|---------|--------------|
| `/gsd:progress` | Where am I? What's next? |
| `/gsd:help` | Show all commands and usage guide |
| `/gsd:update` | Update GSD with changelog preview |
| `/gsd:join-discord` | Join the GSD Discord community |

### Brownfield & Phase Management

| Command | What it does |
|---------|--------------|
| `/gsd:map-codebase` | Analyze existing codebase before new-project |
| `/gsd:add-phase` | Append phase to roadmap |
| `/gsd:insert-phase [N]` | Insert urgent work between phases |
| `/gsd:remove-phase [N]` | Remove future phase, renumber |
| `/gsd:list-phase-assumptions [N]` | See Claude's intended approach before planning |
| `/gsd:plan-milestone-gaps` | Create phases to close gaps from audit |

### Session & Utilities

| Command | What it does |
|---------|--------------|
| `/gsd:pause-work` | Create handoff when stopping mid-phase |
| `/gsd:resume-work` | Restore from last session |
| `/gsd:settings` | Configure model profile and workflow agents |
| `/gsd:set-profile <profile>` | Switch model profile (quality/balanced/budget/eco) |
| `/gsd:add-todo [desc]` | Capture idea for later |
| `/gsd:check-todos` | List pending todos |
| `/gsd:debug [desc]` | Systematic debugging with persistent state |
| `/gsd:quick` | Execute ad-hoc task with GSD guarantees |

---

## The Knowledge Flywheel

Ultra's secret weapon. Each completed phase feeds lessons back into the system:

```
Phase 1 → RETROSPECTIVE → 3 conventions added to CLAUDE.md
Phase 2 → Research swarm reads those conventions → finds patterns 20% faster
Phase 3 → Planner reads 5 conventions + 4 decisions → better first-draft plans
Phase 5 → Verifier checks against conventions → finds issues faster
Phase 8 → Research 60% faster, verification 75% fewer findings
```

**How it works:**

1. `/ultra:retrospective` analyzes completed work
2. Extracts patterns from the code you just built
3. Proposes CLAUDE.md convention additions — you approve each one
4. Logs architectural decisions to STATE.md as `DEC-{NNN}` entries
5. Reviews domain boundaries for shifts

**The math:** Each convention eliminates a class of errors. Five conventions compounding across three phases = exponential improvement. By Phase 8, the system knows your codebase so well that plans converge in 1-2 rounds instead of 5.

Skipping retrospectives costs ~40% of Ultra's long-term value. Don't skip them.

---

## When to Use What

| Criterion | Raw Claude Code | GSD | GSD Ultra |
|-----------|----------------|-----|-----------|
| **Files touched** | 1-2 | 3-10 | 5-30+ |
| **Domains** | 1 | 1-2 | 2-6 |
| **Quality needs** | Prototype | Production | Production+ |
| **Research** | Ad hoc | Single researcher | 4-perspective swarm |
| **Planning** | Inline | Single planner | Builder/Critic debate |
| **Execution** | Single thread | Wave-parallel | Domain-parallel + deferred integration |
| **Verification** | Manual | Single verifier | Triple-check debate |
| **Learning** | None | STATE.md | Full flywheel |
| **Cost** | $0.50-2 | $2-8 | $5-20 |
| **Time** | 5-15 min | 15-45 min | 30-90 min |

**Quick rules:**
- **Bug fix or small feature?** → `/gsd:quick`
- **Standard phase, 1-2 domains?** → GSD workflow
- **Complex phase, 3+ domains, quality critical?** → Ultra pipeline
- **Want maximum quality with zero manual orchestration?** → `/ultra:full-pipeline`

---

## Configuration

GSD stores project settings in `.planning/config.json`. Configure during `/gsd:new-project` or update later with `/gsd:settings`.

### Core Settings

| Setting | Options | Default | What it controls |
|---------|---------|---------|------------------|
| `mode` | `yolo`, `interactive` | `interactive` | Auto-approve vs confirm at each step |
| `depth` | `quick`, `standard`, `comprehensive` | `standard` | Planning thoroughness (phases × plans) |

### Model Profiles

Control which Claude model each agent uses. Balance quality vs token spend.

| Profile | Planning | Execution | Verification | Research | Est. Cost/Phase |
|---------|----------|-----------|--------------|----------|----------------|
| `quality` | Opus | Opus | Sonnet | Opus | ~$27 |
| `balanced` (default) | Opus | Sonnet | Sonnet | Sonnet | ~$13 |
| `budget` | Sonnet | Sonnet | Haiku | Haiku | ~$8 |
| `eco` | Sonnet | Sonnet | Haiku | Haiku | ~$4 |

Switch profiles:
```
/gsd:set-profile eco
```

**Eco** is for well-established projects with strong CLAUDE.md conventions. The knowledge flywheel makes planning and verification easier, so cheaper models still deliver good results. Don't use eco for Phase 1 of a new project.

### Workflow Agents

These spawn additional agents during planning/execution. They improve quality but add tokens and time.

| Setting | Default | What it does |
|---------|---------|--------------|
| `workflow.research` | `true` | Researches domain before planning each phase |
| `workflow.plan_check` | `true` | Verifies plans achieve phase goals before execution |
| `workflow.verifier` | `true` | Confirms must-haves were delivered after execution |

Use `/gsd:settings` to toggle these, or override per-invocation:
- `/gsd:plan-phase --skip-research`
- `/gsd:plan-phase --skip-verify`

### Execution

| Setting | Default | What it controls |
|---------|---------|------------------|
| `parallelization.enabled` | `true` | Run independent plans simultaneously |
| `planning.commit_docs` | `true` | Track `.planning/` in git |

### Git Branching

| Setting | Options | Default | What it does |
|---------|---------|---------|--------------|
| `git.branching_strategy` | `none`, `phase`, `milestone` | `none` | Branch creation strategy |
| `git.phase_branch_template` | string | `gsd/phase-{phase}-{slug}` | Template for phase branches |
| `git.milestone_branch_template` | string | `gsd/{milestone}-{slug}` | Template for milestone branches |

---

## Why It Works

### Context Engineering

Claude Code is incredibly powerful *if* you give it the context it needs. Most people don't.

GSD handles it for you:

| File | What it does |
|------|--------------|
| `PROJECT.md` | Project vision, always loaded |
| `research/` | Ecosystem knowledge (stack, features, architecture, pitfalls) |
| `REQUIREMENTS.md` | Scoped v1/v2 requirements with phase traceability |
| `ROADMAP.md` | Where you're going, what's done |
| `STATE.md` | Decisions, blockers, position — memory across sessions |
| `PLAN.md` | Atomic task with XML structure, verification steps |
| `SUMMARY.md` | What happened, what changed, committed to history |
| `RETROSPECTIVE.md` | Patterns, decisions, conventions — flywheel fuel |

### Adversarial Verification

Single-perspective verification misses things. Ultra's triple-check debate catches what individuals can't:

- **Defender** builds an evidence matrix — requirement → file:line proof
- **Attacker** hunts bugs with composite expertise (security + edge cases + code quality)
- **Auditor** rules on disputes with a weighted compliance score

When all three flag the same issue, it's a **consensus failure** — the strongest signal that something needs fixing.

### Knowledge Compounding

Most AI systems have no memory. They repeat the same mistakes, explore the same dead ends, rediscover the same patterns. Ultra's flywheel breaks this cycle:

```
10% improvement per stage:  1.1 × 1.1 × 1.1 × 1.1 = 1.46× overall
20% improvement per stage:  1.2 × 1.2 × 1.2 × 1.2 = 2.07× overall
```

The multiplicative effect is why skipping stages destroys more value than it saves — you're not losing one stage's benefit, you're losing its compounding effect on everything downstream.

### XML Prompt Formatting

Every plan is structured XML optimized for Claude:

```xml
<task type="auto">
  <name>Create login endpoint</name>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    Use jose for JWT (not jsonwebtoken - CommonJS issues).
    Validate credentials against users table.
    Return httpOnly cookie on success.
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login returns 200 + Set-Cookie</verify>
  <done>Valid credentials return cookie, invalid return 401</done>
</task>
```

Precise instructions. No guessing. Verification built in.

### Multi-Agent Orchestration

Every stage uses the same pattern: a thin orchestrator spawns specialized agents, collects results, and routes to the next step.

The orchestrator never does heavy lifting. It spawns agents, waits, integrates results. **The result:** entire phases run — deep research, adversarial planning, thousands of lines of code across parallel domains, triple-check verification — and your main context window stays at 30-40%.

### Atomic Git Commits

Each task gets its own commit immediately after completion:

```bash
abc123f docs(08-02): complete user registration plan
def456g feat(08-02): add email confirmation flow
hij789k feat(08-02): implement password hashing
```

> [!NOTE]
> **Benefits:** Git bisect finds exact failing task. Each task independently revertable. Clear history for Claude in future sessions.

---

## Security

### Protecting Sensitive Files

GSD's codebase mapping and analysis commands read files to understand your project. **Protect files containing secrets** by adding them to Claude Code's deny list:

1. Open Claude Code settings (`.claude/settings.json` or global)
2. Add sensitive file patterns to the deny list:

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(**/secrets/*)",
      "Read(**/*credential*)",
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ]
  }
}
```

This prevents Claude from reading these files entirely, regardless of what commands you run.

> [!IMPORTANT]
> GSD includes built-in protections against committing secrets, but defense-in-depth is best practice. Deny read access to sensitive files as a first line of defense.

---

## Troubleshooting

**Commands not found after install?**
- Restart Claude Code to reload slash commands
- Verify files exist in `~/.claude/commands/gsd/` (global) or `./.claude/commands/gsd/` (local)

**Commands not working as expected?**
- Run `/gsd:help` to verify installation
- Re-run `npx get-shit-done-cc` to reinstall

**Updating to the latest version?**
```bash
npx get-shit-done-cc@latest
```

**Using Docker or containerized environments?**

If file reads fail with tilde paths (`~/.claude/...`), set `CLAUDE_CONFIG_DIR` before installing:
```bash
CLAUDE_CONFIG_DIR=/home/youruser/.claude npx get-shit-done-cc --global
```

### Uninstalling

```bash
# Global installs
npx get-shit-done-cc --claude --global --uninstall
npx get-shit-done-cc --opencode --global --uninstall

# Local installs (current project)
npx get-shit-done-cc --claude --local --uninstall
npx get-shit-done-cc --opencode --local --uninstall
```

---

## Community Ports

OpenCode and Gemini CLI are now natively supported via `npx get-shit-done-cc`.

| Project | Platform | Description |
|---------|----------|-------------|
| [gsd-opencode](https://github.com/rokicool/gsd-opencode) | OpenCode | Original OpenCode adaptation |
| gsd-gemini (archived) | Gemini CLI | Original Gemini adaptation by uberfuzzy |

---

## Star History

<a href="https://star-history.com/#bslevin87/get-shit-done_ultra&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=bslevin87/get-shit-done_ultra&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=bslevin87/get-shit-done_ultra&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=bslevin87/get-shit-done_ultra&type=Date" />
 </picture>
</a>

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**Claude Code is powerful. GSD makes it reliable. Ultra makes it extraordinary.**

</div>
