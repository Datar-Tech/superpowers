# Agent Teams + Superpowers Integration Guide

> How to use Superpowers skills effectively within Claude Code Agent Teams.

## Background

Claude Code Agent Teams (experimental) lets multiple Claude Code instances collaborate as a team — a Lead orchestrates work, Teammates execute independently, and they communicate via shared task lists and messaging.

Superpowers provides 14 reusable skills covering software engineering discipline (TDD, debugging, verification) and workflow orchestration (subagent dispatch, plan execution).

These two systems overlap in orchestration but complement each other in methodology. This guide explains how to combine them effectively.

## Core Principle

```
Agent Teams  → "who does what"  (orchestration, parallelism, communication)
Superpowers  → "how to do it"   (discipline, methodology, quality)
```

Let each layer do what it's best at. Don't mix orchestration mechanisms.

## Skill Compatibility Matrix

### Use normally (10 skills)

These skills work in Agent Teams without modification. Teammates auto-load them.

| Skill | Who uses it | How |
|-------|------------|-----|
| `using-superpowers` | Lead + all Teammates | Auto-loaded, works normally |
| `brainstorming` | Lead | Collaborative design with user, before spawning teammates |
| `writing-plans` | Lead | Produces plan document as basis for task breakdown |
| `writing-skills` | Lead | When developing new skills |
| `test-driven-development` | Teammates | Each implementer follows Red-Green-Refactor |
| `systematic-debugging` | Teammates | Root cause analysis before fixes |
| `verification-before-completion` | Teammates + Lead | Verify before claiming completion |
| `receiving-code-review` | Teammates | Evaluate review feedback with technical rigor |
| `using-git-worktrees` | Lead | Create isolated workspace once before spawning |
| `finishing-a-development-branch` | Lead | Wrap up after all teammates finish |

### Replaced by Agent Teams (4 skills)

These skills use the `Task` tool for orchestration, which Agent Teams handles natively and better (teammates can communicate with each other).

| Skill | Replaced by |
|-------|------------|
| `subagent-driven-development` | Lead spawns implementer + reviewer teammates |
| `dispatching-parallel-agents` | Lead spawns multiple teammates in parallel |
| `executing-plans` | Lead manages shared task list with checkpoints |
| `requesting-code-review` | Dedicated reviewer teammate (can dialogue with implementer) |

**Do not use these 4 skills when Agent Teams is active.** They would create nested orchestration (Agent Teams -> Teammate -> Subagents), causing confusion and wasted tokens.

## Best Practices

### 1. Write explicit spawn prompts

Teammates auto-load skills but don't inherit the Lead's conversation history. Explicitly reference discipline skills in spawn prompts:

**Good:**
```
Implement the JWT verification module in src/auth/.
Requirements:
- Follow test-driven-development (write tests before code)
- Run verification-before-completion before marking done
- If you hit bugs, use systematic-debugging for root cause analysis
- Do NOT use subagent-driven-development or dispatching-parallel-agents
Spec: docs/plans/2026-02-12-auth-design.md
```

**Bad:**
```
Implement JWT verification
```

### 2. Prevent nested orchestration

Teammates may accidentally trigger orchestration skills if their task is broad enough. Prevent this by:

1. **Explicitly exclude in spawn prompt**: "Do not use subagent-driven-development or dispatching-parallel-agents — your work is coordinated by the Agent Teams Lead."
2. **Keep tasks small enough** that teammates don't feel the need to sub-orchestrate.

### 3. Use reviewer teammates instead of requesting-code-review

Replace the `requesting-code-review` subagent dispatch with a dedicated reviewer teammate:

```
You are a code reviewer. Review commits from implementer teammates.
- Use git diff main...HEAD to see changes
- Check: test coverage, security, performance
- Follow receiving-code-review principles (no performative agreement)
- Message implementers directly about issues
- Critical issues must be fixed before task completion
```

**Advantage:** The reviewer and implementer can have a back-and-forth dialogue, which is stronger than Superpowers' one-shot subagent review.

### 4. Lead handles the workflow envelope

The Lead should own the Superpowers workflow stages that happen before and after parallel work:

```
Lead: brainstorming (with user)
  → writing-plans (produce plan)
  → using-git-worktrees (create workspace)
  → [spawn teammates — Agent Teams takes over orchestration]
  → [teammates use TDD, debugging, verification autonomously]
  → finishing-a-development-branch (wrap up)
```

## Recommended Team Templates

### Template A: Feature Development (3-5 members)

```
Lead: brainstorming → writing-plans → break into tasks
  ├── Implementer A (module 1) → TDD + verification
  ├── Implementer B (module 2) → TDD + verification
  ├── Reviewer (reviews all commits) → receiving-code-review discipline
  └── Lead synthesizes → finishing-a-development-branch
```

### Template B: Debugging with Competing Hypotheses (3 members)

```
Lead: describe symptoms
  ├── Investigator A (hypothesis 1) → systematic-debugging
  ├── Investigator B (hypothesis 2) → systematic-debugging
  ├── Investigator C (hypothesis 3) → systematic-debugging
  └── Investigators message each other to challenge theories
Lead: synthesize consensus → verification-before-completion
```

### Template C: Large-Scale Refactoring (4+ members)

```
Lead: writing-plans → split by module
  ├── Implementer × N (each owns independent module) → TDD
  ├── Integration Tester (cross-module tests) → verification
  └── Lead: finishing-a-development-branch
```

## Why Not an Adapter Skill?

We considered creating a `using-agent-teams` skill to automatically bridge the two systems. This is not feasible because:

1. **Skills are Markdown instructions** — they cannot programmatically detect the Agent Teams environment, intercept tool calls, or override other skills
2. **No skill priority mechanism** — if two skills give contradictory instructions (e.g., "use Task tool" vs. "use teammate messaging"), behavior is unpredictable
3. **Teammate context isolation** — a skill loaded by the Lead cannot influence teammate behavior; teammates have their own context
4. **Unstable API** — Agent Teams is experimental; hardcoding tool names in a skill is fragile

The manual approach described in this guide is more reliable and maintainable.

## When Agent Teams Stabilizes

Once Agent Teams exits experimental status, consider these lightweight changes to Superpowers:

1. Add exclusion hints to orchestration skill descriptions — e.g., `subagent-driven-development`: "Do not use when running as an Agent Teams teammate"
2. Add Agent Teams awareness to `using-superpowers` — a short paragraph guiding the Lead to skip orchestration skills when Agent Teams is active
3. Keep this guide updated as the Agent Teams API evolves

## References

- [Claude Code Agent Teams documentation](https://code.claude.com/docs/en/agent-teams)
- [Superpowers Skills README](../README.md)
