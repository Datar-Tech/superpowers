 Agent Teams + Superpowers Integration Guide 說明

  這份文件說明如何在 Claude Code Agent Teams（實驗性功能）中有效使用 Superpowers skills。

  核心概念

  兩套系統各司其職，不要混用：

  - Agent Teams → 負責「誰做什麼」（協調、平行處理、溝通）
  - Superpowers → 負責「怎麼做」（紀律、方法論、品質）

  Skills 相容性

  10 個 skills 可正常使用，包括：
  - brainstorming、writing-plans → Lead 使用
  - test-driven-development、systematic-debugging、verification-before-completion → Teammates 使用
  - finishing-a-development-branch、using-git-worktrees → Lead 使用

  4 個 skills 被 Agent Teams 取代，在 Agent Teams 啟動時不應使用：
  - subagent-driven-development → 改由 Lead 生成 teammates
  - dispatching-parallel-agents → 改由 Lead 平行生成多個 teammates
  - executing-plans → 改由 Lead 管理共享任務列表
  - requesting-code-review → 改用專屬 reviewer teammate

  原因：這 4 個 skill 都用 Task tool 做協調，會造成「巢狀協調」（Agent Teams → Teammate → Subagents），浪費 tokens 且混亂。

  最佳實踐

  1. 寫明確的 spawn prompt — Teammates 不繼承 Lead 的對話歷史，要明確指定使用哪些 skills
  2. 防止巢狀協調 — 在 spawn prompt 中明確排除協調類 skills
  3. 用 reviewer teammate 取代 requesting-code-review — 好處是 reviewer 和 implementer 能來回對話
  4. Lead 負責工作流的首尾 — brainstorming → writing-plans → worktrees → [生成 teammates] → finishing

  三種推薦團隊模板
  ┌────────────────────────┬────────────────────┬──────────────────────────────────────────────┐
  │          模板          │        用途        │                     成員                     │
  ├────────────────────────┼────────────────────┼──────────────────────────────────────────────┤
  │ A: Feature Development │ 功能開發           │ Lead + 2 Implementers + 1 Reviewer           │
  ├────────────────────────┼────────────────────┼──────────────────────────────────────────────┤
  │ B: Debugging           │ 除錯（競爭假設法） │ Lead + 3 Investigators                       │
  ├────────────────────────┼────────────────────┼──────────────────────────────────────────────┤
  │ C: Large Refactoring   │ 大規模重構         │ Lead + N Implementers + 1 Integration Tester │
  └────────────────────────┴────────────────────┴──────────────────────────────────────────────┘
  為什麼不做 Adapter Skill？

  文件解釋了不做自動橋接 skill 的原因：Skills 是 Markdown 指令，無法偵測環境或攔截 tool calls；且 Agent Teams API 仍不穩定。

  ---

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
