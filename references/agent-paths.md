# QClaw / OpenClaw 知识文件路径速查

执行第一步盘点时按这张表找文件。

## Workspace 知识体系

| 用途 | 路径 |
|---|---|
| Agent 人格 | `<workspace>/SOUL.md` + `<workspace>/IDENTITY.md` |
| 长期记忆 | `<workspace>/MEMORY.md` |
| 工作日志 | `<workspace>/memory/YYYY-MM-DD.md` |
| 日志归档 | `<workspace>/memory/archive/` |
| 任务摘要 | `<workspace>/task-summary_*.md` |
| Agent 配置 | `<workspace>/AGENTS.md` |
| 用户信息 | `<workspace>/USER.md` |
| 工具备注 | `<workspace>/TOOLS.md` |
| 心跳任务 | `<workspace>/HEARTBEAT.md` |

`<workspace>` 通常是 `~/.qclaw/workspace-agent-<id>` 或 `~/.openclaw/workspace`。

## self-improving 记忆体系

| 用途 | 路径 |
|---|---|
| HOT 规则（≤100行） | `~/self-improving/memory.md` |
| 纠正日志 | `~/self-improving/corrections.md` |
| 索引文件 | `~/self-improving/index.md` |
| 项目知识 | `~/self-improving/projects/` |
| 领域知识 | `~/self-improving/domains/` |
| 归档 | `~/self-improving/archive/` |
| 心跳状态 | `~/self-improving/heartbeat-state.md` |

## Skills 目录

| 用途 | 路径 |
|---|---|
| 用户级 skills | `~/.qclaw/skills/<name>/SKILL.md` |
| 内置 skills | `~/Library/Application Support/QClaw/openclaw/config/skills/` |
| Workspace skills | `<workspace>/skills/` |

## LCM

LCM（Lossless Context Management）的 compacted summary 不可直接编辑。审查方式：

- `lcm_grep` — 搜索关键实体
- `lcm_describe` — 查看特定摘要
- `lcm_expand_query` — 深度展开摘要内容

发现过期信息 → 在 MEMORY.md 中写覆写条目。
