# Agent Teams Skill Modifications Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add Agent Teams awareness to 5 Superpowers skills so they self-document when NOT to use them, preventing nested orchestration in Agent Teams environments.

**Architecture:** Add exclusion hints to 4 orchestration skills (frontmatter description + body warning section) and add an Agent Teams awareness paragraph to `using-superpowers`. No behavior changes — documentation-only modifications.

**Tech Stack:** Markdown (SKILL.md files)

---

### Task 1: Add Agent Teams exclusion to `subagent-driven-development`

**Files:**
- Modify: `skills/subagent-driven-development/SKILL.md:1-4` (frontmatter)
- Modify: `skills/subagent-driven-development/SKILL.md:12` (after "When to Use" section heading, add warning)

**Step 1: Update frontmatter description**

Change line 3 from:
```yaml
description: Use when executing implementation plans with independent tasks in the current session
```
to:
```yaml
description: Use when executing implementation plans with independent tasks in the current session. Do NOT use when running as an Agent Teams teammate — Agent Teams handles orchestration natively.
```

**Step 2: Add Agent Teams warning after the opening paragraph**

Insert after line 10 (after "Fresh subagent per task + two-stage review...") a new section:

```markdown

> **Agent Teams:** Do NOT use this skill when Claude Code Agent Teams is active. Agent Teams replaces this skill's orchestration — the Lead spawns implementer + reviewer teammates instead. Using both creates nested orchestration (Agent Teams → Teammate → Subagents), wasting tokens and causing confusion. See `docs/agent-teams-integration-guide.md`.
```

**Step 3: Verify the file renders correctly**

Run: `head -15 skills/subagent-driven-development/SKILL.md`
Expected: Frontmatter has updated description, warning block visible after intro.

**Step 4: Commit**

```bash
git add skills/subagent-driven-development/SKILL.md
git commit -m "feat(skills): add Agent Teams exclusion hint to subagent-driven-development"
```

---

### Task 2: Add Agent Teams exclusion to `dispatching-parallel-agents`

**Files:**
- Modify: `skills/dispatching-parallel-agents/SKILL.md:1-4` (frontmatter)
- Modify: `skills/dispatching-parallel-agents/SKILL.md:12` (after "Core principle" line, add warning)

**Step 1: Update frontmatter description**

Change line 3 from:
```yaml
description: Use when facing 2+ independent tasks that can be worked on without shared state or sequential dependencies
```
to:
```yaml
description: Use when facing 2+ independent tasks that can be worked on without shared state or sequential dependencies. Do NOT use when running as an Agent Teams teammate — Agent Teams handles parallel dispatch natively.
```

**Step 2: Add Agent Teams warning after the "Core principle" line**

Insert after line 12 (after "Dispatch one agent per independent problem domain..."):

```markdown

> **Agent Teams:** Do NOT use this skill when Claude Code Agent Teams is active. Agent Teams replaces this skill — the Lead spawns multiple teammates in parallel instead. Using both creates nested orchestration. See `docs/agent-teams-integration-guide.md`.
```

**Step 3: Verify the file renders correctly**

Run: `head -18 skills/dispatching-parallel-agents/SKILL.md`
Expected: Updated description and warning block visible.

**Step 4: Commit**

```bash
git add skills/dispatching-parallel-agents/SKILL.md
git commit -m "feat(skills): add Agent Teams exclusion hint to dispatching-parallel-agents"
```

---

### Task 3: Add Agent Teams exclusion to `executing-plans`

**Files:**
- Modify: `skills/executing-plans/SKILL.md:1-4` (frontmatter)
- Modify: `skills/executing-plans/SKILL.md:14` (after "Announce at start" line, add warning)

**Step 1: Update frontmatter description**

Change line 3 from:
```yaml
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
```
to:
```yaml
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints. Do NOT use when running as an Agent Teams teammate — the Lead manages the shared task list with checkpoints natively.
```

**Step 2: Add Agent Teams warning after the "Announce at start" line**

Insert after line 14 (after the announce line):

```markdown

> **Agent Teams:** Do NOT use this skill when Claude Code Agent Teams is active. Agent Teams replaces this skill — the Lead manages a shared task list with checkpoints instead. Using both creates nested orchestration. See `docs/agent-teams-integration-guide.md`.
```

**Step 3: Verify the file renders correctly**

Run: `head -20 skills/executing-plans/SKILL.md`
Expected: Updated description and warning block visible.

**Step 4: Commit**

```bash
git add skills/executing-plans/SKILL.md
git commit -m "feat(skills): add Agent Teams exclusion hint to executing-plans"
```

---

### Task 4: Add Agent Teams exclusion to `requesting-code-review`

**Files:**
- Modify: `skills/requesting-code-review/SKILL.md:1-4` (frontmatter)
- Modify: `skills/requesting-code-review/SKILL.md:10` (after "Core principle" line, add warning)

**Step 1: Update frontmatter description**

Change line 3 from:
```yaml
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
```
to:
```yaml
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements. Do NOT use when running as an Agent Teams teammate — use a dedicated reviewer teammate instead.
```

**Step 2: Add Agent Teams warning after the "Core principle" line**

Insert after line 10 (after "Review early, review often."):

```markdown

> **Agent Teams:** Do NOT use this skill when Claude Code Agent Teams is active. Instead, spawn a dedicated reviewer teammate who can have a back-and-forth dialogue with the implementer — stronger than one-shot subagent review. See `docs/agent-teams-integration-guide.md`.
```

**Step 3: Verify the file renders correctly**

Run: `head -16 skills/requesting-code-review/SKILL.md`
Expected: Updated description and warning block visible.

**Step 4: Commit**

```bash
git add skills/requesting-code-review/SKILL.md
git commit -m "feat(skills): add Agent Teams exclusion hint to requesting-code-review"
```

---

### Task 5: Add Agent Teams awareness to `using-superpowers`

**Files:**
- Modify: `skills/using-superpowers/SKILL.md:68` (before "Skill Priority" section, add new section)

**Step 1: Add "Agent Teams Compatibility" section**

Insert before line 67 (`## Skill Priority`), a new section:

```markdown
## Agent Teams Compatibility

When Claude Code Agent Teams is active (Lead + Teammates pattern), **skip the 4 orchestration skills** — Agent Teams handles orchestration natively and better:

| Skip this skill | Agent Teams replacement |
|----------------|----------------------|
| `subagent-driven-development` | Lead spawns implementer + reviewer teammates |
| `dispatching-parallel-agents` | Lead spawns multiple teammates in parallel |
| `executing-plans` | Lead manages shared task list with checkpoints |
| `requesting-code-review` | Dedicated reviewer teammate (can dialogue) |

**All other skills (10 of 14) work normally** in Agent Teams. Teammates should especially use: `test-driven-development`, `systematic-debugging`, `verification-before-completion`.

See `docs/agent-teams-integration-guide.md` for full details, spawn prompt templates, and team templates.

```

**Step 2: Verify the file renders correctly**

Run: `grep -A 5 "Agent Teams" skills/using-superpowers/SKILL.md`
Expected: New section visible with compatibility table.

**Step 3: Commit**

```bash
git add skills/using-superpowers/SKILL.md
git commit -m "feat(skills): add Agent Teams awareness to using-superpowers"
```

---

### Task 6: Final verification

**Step 1: Verify all 5 files were modified**

Run: `git diff --stat main`
Expected: 5 files changed (the 5 SKILL.md files listed above).

**Step 2: Grep for consistency**

Run: `grep -r "Agent Teams" skills/*/SKILL.md`
Expected: Hits in all 5 modified files. No hits in other skills.

**Step 3: Verify no broken frontmatter**

Run: `head -4 skills/subagent-driven-development/SKILL.md skills/dispatching-parallel-agents/SKILL.md skills/executing-plans/SKILL.md skills/requesting-code-review/SKILL.md`
Expected: All start with `---`, have `name:` and `description:`, end with `---`.

**Step 4: Commit (if any fixups needed)**

Only if previous steps revealed issues. Otherwise skip.
