# QClaw / OpenClaw 知识文件路径速查

执行第一步盘点时按这张表找文件。Agent 根据当前系统（系统提示中 `os=` 字段）选择对应列。

## Workspace 知识体系

| 用途 | macOS / Linux | Windows |
|---|---|---|
| Agent 人格 | `<workspace>/SOUL.md` + `<workspace>/IDENTITY.md` | 同左 |
| 长期记忆 | `<workspace>/MEMORY.md` | 同左 |
| 工作日志 | `<workspace>/memory/YYYY-MM-DD.md` | 同左 |
| 日志归档 | `<workspace>/memory/archive/` | 同左 |
| 任务摘要 | `<workspace>/task-summary_*.md` | 同左 |
| Agent 配置 | `<workspace>/AGENTS.md` | 同左 |
| 用户信息 | `<workspace>/USER.md` | 同左 |
| 工具备注 | `<workspace>/TOOLS.md` | 同左 |
| 心跳任务 | `<workspace>/HEARTBEAT.md` | 同左 |

`<workspace>` 路径：

| 系统 | 路径 |
|---|---|
| macOS / Linux | `~/.qclaw/workspace-agent-<id>` 或 `~/.openclaw/workspace` |
| Windows | `%USERPROFILE%\.qclaw\workspace-agent-<id>` 或 `%USERPROFILE%\.openclaw\workspace` |

## self-improving 记忆体系

| 用途 | macOS / Linux | Windows |
|---|---|---|
| HOT 规则（≤100行） | `~/self-improving/memory.md` | `%USERPROFILE%\self-improving\memory.md` |
| 纠正日志 | `~/self-improving/corrections.md` | `%USERPROFILE%\self-improving\corrections.md` |
| 索引文件 | `~/self-improving/index.md` | `%USERPROFILE%\self-improving\index.md` |
| 项目知识 | `~/self-improving/projects/` | `%USERPROFILE%\self-improving\projects\` |
| 领域知识 | `~/self-improving/domains/` | `%USERPROFILE%\self-improving\domains\` |
| 归档 | `~/self-improving/archive/` | `%USERPROFILE%\self-improving\archive\` |
| 心跳状态 | `~/self-improving/heartbeat-state.md` | `%USERPROFILE%\self-improving\heartbeat-state.md` |

## Skills 目录

| 用途 | macOS / Linux | Windows |
|---|---|---|
| 用户级 skills | `~/.qclaw/skills/<name>/SKILL.md` | `%USERPROFILE%\.qclaw\skills\<name>\SKILL.md` |
| 内置 skills | `~/Library/Application Support/QClaw/openclaw/config/skills/` | `%APPDATA%\QClaw\openclaw\config\skills\` |
| Workspace skills | `<workspace>/skills/` | 同左 |

## LCM

LCM（Lossless Context Management）的 compacted summary 不可直接编辑。审查方式：

- `lcm_grep` — 搜索关键实体
- `lcm_describe` — 查看特定摘要
- `lcm_expand_query` — 深度展开摘要内容

发现过期信息 → 在 MEMORY.md 中写覆写条目。

## 自检清单中的 Shell 命令适配

SKILL.md 自检清单中的 `grep -E` 命令，按系统替换：

| 系统 | 命令 |
|---|---|
| macOS / Linux | `grep -E "今天\|昨天\|刚刚\|最近\|上周\|today\|yesterday\|recently"` |
| Windows (PowerShell) | `Select-String -Pattern "今天\|昨天\|刚刚\|最近\|上周\|today\|yesterday\|recently"` |
| Windows (Git Bash) | 同 macOS / Linux |