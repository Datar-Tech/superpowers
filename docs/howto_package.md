● Superpowers 如何打包為 Claude Code Plugin

  Superpowers 已經是一個 Claude Code plugin。以下是它的打包方式：

  ---
  目錄結構（必要部分）

  your-plugin/
  ├── .claude-plugin/
  │   ├── plugin.json          # 插件元資料（名稱、版本、作者）
  │   └── marketplace.json     # 市集定義（如果要發布到 marketplace）
  ├── hooks/
  │   ├── hooks.json           # Hook 定義（SessionStart 等）
  │   ├── session-start.sh     # Bootstrap 腳本
  │   └── run-hook.cmd         # Windows 相容層
  ├── skills/                  # 技能目錄
  │   └── my-skill/
  │       └── SKILL.md         # 技能定義（YAML frontmatter + 內容）
  ├── commands/                # Slash commands（可選）
  │   └── my-command.md
  └── agents/                  # Agent 定義（可選）
      └── my-agent.md

  三個關鍵檔案

  1. .claude-plugin/plugin.json — 插件身份：
  {
    "name": "superpowers",
    "description": "Core skills library for Claude Code",
    "version": "4.2.0",
    "author": { "name": "Jesse Vincent" },
    "repository": "https://github.com/obra/superpowers",
    "license": "MIT",
    "keywords": ["skills", "tdd", "debugging"]
  }

  2. hooks/hooks.json — 啟動時注入：
  {
    "hooks": {
      "SessionStart": [{
        "matcher": "startup|resume|clear|compact",
        "hooks": [{
          "type": "command",
          "command": "${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh",
          "async": true
        }]
      }]
    }
  }

  3. skills/*/SKILL.md — 每個技能一個：
  ---
  name: my-skill
  description: "Use when [觸發條件描述]"
  ---
  # 技能內容
  [Markdown 指令]

  安裝方式

  有兩種方式安裝 Claude Code plugin：

  方式 A：Marketplace（推薦）
  # 加入 marketplace
  claude /plugin marketplace add obra/superpowers-marketplace

  # 安裝 plugin
  claude /plugin install superpowers@superpowers-marketplace

  方式 B：本地安裝
  # 直接從本地路徑安裝
  claude /plugin install /path/to/your-plugin

  重點概念
  ┌──────────┬───────────────────────────────────────────────────────────┐
  │   概念   │                           說明                            │
  ├──────────┼───────────────────────────────────────────────────────────┤
  │ Skills   │ 放在 skills/ 下，用 SKILL.md 的 YAML frontmatter 自動發現 │
  ├──────────┼───────────────────────────────────────────────────────────┤
  │ Hooks    │ hooks.json 定義生命週期事件（如 SessionStart）            │
  ├──────────┼───────────────────────────────────────────────────────────┤
  │ Commands │ commands/ 下的 .md 檔案變成 slash commands                │
  ├──────────┼───────────────────────────────────────────────────────────┤
  │ Agents   │ agents/ 下的 .md 檔案定義 subagent 類型                   │
  ├──────────┼───────────────────────────────────────────────────────────┤
  │ 更新     │ 基於 git — git pull 即可，symlink 讓更新即時生效          │
  └──────────┴───────────────────────────────────────────────────────────┘
  ---
  簡單來說，Claude Code plugin 就是一個有特定目錄結構的 git repo，不需要 npm/build，只要 JSON 元資料 + Markdown 技能檔就能運作。