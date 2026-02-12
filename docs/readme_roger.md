# Roger's Fork 維護筆記

## 分支策略

```
main   → 永遠與 upstream (obra/superpowers) 保持一致，不放自訂內容
custom → upstream + 我的修改，日常使用這個分支
```

### 同步上游

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

git checkout custom
git rebase main
git push origin custom --force-with-lease
```

### 新增自訂內容

所有自訂修改都 commit 到 `custom` 分支。不要直接改 `main`。

## Agent Teams + Superpowers 整合注意事項

詳見 [agent-teams-integration-guide.md](agent-teams-integration-guide.md)

### 重點摘要

**10 個技能正常使用（Agent Teams 環境也相容）：**
- `using-superpowers`、`brainstorming`、`writing-plans`、`writing-skills`
- `test-driven-development`、`systematic-debugging`、`verification-before-completion`
- `receiving-code-review`、`using-git-worktrees`、`finishing-a-development-branch`

**4 個技能在 Agent Teams 環境中應跳過（由原生機制取代）：**
- `subagent-driven-development` → Lead spawn teammates
- `dispatching-parallel-agents` → Lead spawn teammates in parallel
- `executing-plans` → Lead + shared task list
- `requesting-code-review` → Dedicated reviewer teammate

### 為什麼不做「適配技能」

- Skill 是 Markdown 指令，無法偵測環境、攔截 tool calls、覆蓋其他技能
- 技能間無優先級機制，矛盾指令會導致不可預測行為
- Agent Teams 仍是實驗功能，API 不穩定

### 等 Agent Teams 穩定後的輕量調整

1. 在編排技能的 description 加排除條件（如 "Do not use when running as Agent Teams teammate"）
2. 在 `using-superpowers` 加一段 Agent Teams 感知指引
3. 持續更新整合指南

## 自訂文件清單

| 檔案 | 用途 |
|------|------|
| `docs/agent-teams-integration-guide.md` | Agent Teams 搭配 Superpowers 的最佳實踐 |
| `docs/readme_roger.md` | 本檔案，fork 維護筆記 |
