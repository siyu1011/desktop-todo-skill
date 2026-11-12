# desktop-todo-skill

给各种 coding agent 用的通用 [Agent Skill](https://agentskills.io)，用来在你自己部署的待办服务里增删改查待办，并同步到桌面小组件。

适用于 Codex、Claude Code、Cursor、OpenCode、Gemini CLI 等能加载 Agent Skills 的工具。不绑定某一个产品。

只要本机有 **Python 3**（只用标准库，无需再装依赖）。Windows、macOS、Linux 都可以。

## 安装

推荐用 [skills CLI](https://github.com/vercel-labs/skills) 一键安装（会问要装到哪些 agent；加 `-g` 装到当前用户，所有项目都能用）：

```bash
npx skills add siyu1011/desktop-todo-skill
# 或者全局安装：
npx skills add siyu1011/desktop-todo-skill -g
```

也可以手动把仓库里的 `desktop-todo/` 文件夹复制到对应目录（目录名保持 `desktop-todo`，里面要有 `SKILL.md`）：

| Agent | 用户级（全局） | 项目级 |
| --- | --- | --- |
| Codex | `~/.agents/skills/`（推荐；旧路径 `~/.codex/skills/` 仍可用） | `.agents/skills/` |
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Cursor | `~/.cursor/skills/` | `.agents/skills/` |
| OpenCode | `~/.config/opencode/skills/` | `.opencode/skills/` |
| Gemini CLI | `~/.gemini/skills/` | `.agents/skills/` |

例如 Codex 全局安装：

```bash
git clone https://github.com/siyu1011/desktop-todo-skill.git
cp -R desktop-todo-skill/desktop-todo ~/.agents/skills/desktop-todo
```

以上路径按 2026 年各官方文档核对过：Codex 现在推荐 `~/.agents/skills/`（`~/.codex/skills/` 仅作兼容）；OpenCode 全局目录是 `~/.config/opencode/skills/`（复数 `skills`）。

## 配置

1. 安装待办桌面小组件，并连接到你的待办服务器。
2. 在待办服务器上注册一个账号（可在小组件里完成）。
3. 打开小组件 **设置 → AI 绑定密钥**，显示并复制密钥。重新生成后，旧密钥会立即失效。
4. 设置环境变量（把地址换成**你的待办服务器地址**）：

```bash
export TODO_API_URL='https://your-todo-server.example:8787'
export TODO_API_KEY='todo_你的密钥'
```

也可以写入 `~/.config/todo/config.json`（权限建议 `600`），或用 `TODO_CLI_CONFIG` 指定别的路径：

```json
{
  "url": "https://your-todo-server.example:8787",
  "api_key": "todo_你的密钥"
}
```

不要把密钥提交到 git，也不要发给无关的人。Agent 调用出错且退出码为 3 时，到 **设置 → AI 绑定密钥** 复制新密钥即可。

验证：

```bash
python3 /path/to/desktop-todo/scripts/todo whoami
python3 /path/to/desktop-todo/scripts/todo list --json
```

Windows 若没有 `python3` 命令，改用 `python`。

## 仓库结构

```
desktop-todo/
  SKILL.md          # agent 读取的说明
  scripts/todo      # 纯 Python 3 命令行（标准库）
```

---

## English

A universal [Agent Skill](https://agentskills.io) for a self-hosted todo service and its desktop widget. Works with Codex, Claude Code, Cursor, OpenCode, Gemini CLI, and other agents that load the Agent Skills layout.

**Requires Python 3** (stdlib only).

Install:

```bash
npx skills add siyu1011/desktop-todo-skill
# or globally:
npx skills add siyu1011/desktop-todo-skill -g
```

Or copy the `desktop-todo/` folder into the agent’s skills directory:

- Codex: `~/.agents/skills/` (preferred; legacy `~/.codex/skills/` still works)
- Claude Code: `~/.claude/skills/`
- Cursor: `~/.cursor/skills/` (project: `.agents/skills/`)
- OpenCode: `~/.config/opencode/skills/` (project: `.opencode/skills/`)
- Gemini CLI: `~/.gemini/skills/`

Then set `TODO_API_URL` to **your todo server address** and `TODO_API_KEY` to the key from the widget: **设置 → AI 绑定密钥**. Regenerating the key invalidates the old one immediately. You can also put `url` and `api_key` in `~/.config/todo/config.json`.
