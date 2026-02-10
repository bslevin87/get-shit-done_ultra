# Quickstart

Get GSD Ultra running in under 5 minutes.

## Install

```bash
npx get-shit-done-cc@latest
```

Choose your runtime (Claude Code, OpenCode, Gemini) and location (global or local).

## First Run

```
/gsd:help
```

If you see the command list, you're installed. Ultra commands appear under the `ultra:` namespace.

## Your First Project

```
/gsd:new-project
```

Answer questions about your idea. The system creates `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, and `STATE.md`.

## Your First Phase (GSD)

```
/gsd:discuss-phase 1    # Shape implementation decisions
/gsd:plan-phase 1       # Research + plan + verify
/gsd:execute-phase 1    # Build in parallel waves
/gsd:verify-work 1      # Walk through deliverables
```

## Your First Phase (Ultra)

For complex phases with multiple domains:

```
/ultra:research-swarm 1      # 4-perspective research
/ultra:adversarial-plan 1    # Builder/Critic debate
/ultra:parallel-execute 1    # Domain-parallel execution
/ultra:adversarial-verify 1  # Triple-check debate
/ultra:gap-close 1           # Fix what verification found
/ultra:retrospective 1       # Learn from the phase
```

Or chain everything:

```
/ultra:full-pipeline 1
```

## Ultra Setup (Optional)

For multi-domain projects, create `gsd-ultra.json` in your project root:

```json
{
  "domains": {
    "auth": { "paths": ["src/features/auth/"] },
    "tasks": { "paths": ["src/features/tasks/"] }
  },
  "shared_paths": ["src/types/", "src/lib/"]
}
```

Without this file, Ultra auto-discovers domains from your codebase.

## Dry-Run Checklist

Before using Ultra on a real project, verify:

- [ ] `/gsd:help` shows all commands (both `gsd:` and `ultra:`)
- [ ] `.planning/config.json` exists after first command
- [ ] `gsd-ultra.json` is recognized (if created)
- [ ] `/gsd:set-profile balanced` confirms profile switch
- [ ] Git is initialized in your project directory

## Recommended Settings

```bash
# Run Claude Code with skip-permissions for frictionless automation
claude --dangerously-skip-permissions
```

Set model profile based on your needs:

```
/gsd:set-profile quality     # Maximum reasoning power
/gsd:set-profile balanced    # Smart allocation (default)
/gsd:set-profile budget      # Minimal Opus usage
/gsd:set-profile eco         # Maximum efficiency
```

## Next Steps

- [Pipeline Overview](01-pipeline-overview.md) — understand the 6-stage pipeline
- [Agents](02-agents.md) — meet the 20 agents
- [Configuration and Cost](08-configuration-and-cost.md) — optimize for your budget
