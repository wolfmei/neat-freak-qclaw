# Hermes Agent 知识文件路径速查

执行第一步盘点时按这张表找文件。

## 核心知识文件

| 用途 | 路径 | 说明 |
|---|---|---|
| Agent 人格 | $HERMES_HOME/SOUL.md | 全局唯一，不按项目隔离 |
| 长期记忆 | ~/.hermes/memories/MEMORY.md | **2200 字符上限**，§ 分隔条目 |
| 用户画像 | ~/.hermes/memories/USER.md | **1375 字符上限**，§ 分隔条目 |
| 项目上下文 | <cwd>/.hermes.md 或 AGENTS.md 或 CLAUDE.md | 优先级：.hermes.md > AGENTS.md > CLAUDE.md > .cursorrules |
| 子目录上下文 | <subdir>/AGENTS.md | 渐进式发现：agent 进入子目录时自动加载 |

`HERMES_HOME` 默认为 `~/.hermes`。

## 项目上下文优先级

同一会话只加载一个项目上下文（first match wins）：

1. `.hermes.md` — Hermes 原生，最高优先级
2. `AGENTS.md` — 通用格式
3. `CLAUDE.md` — Claude Code 兼容
4. `.cursorrules` — Cursor IDE 兼容

## Skills 目录

| 用途 | 路径 |
|---|---|
| 本地 skills | ~/.hermes/skills/<name>/SKILL.md |
| 外部 skill 目录 | 配置文件中指定的额外目录 |

Skills 使用 [agentskills.io](https://agentskills.io) 开放标准。

## 会话搜索

Hermes 内置 FTS5 全文搜索，搜索历史会话内容：

- 通过 Hermes 的 session search 工具访问
- 用于回溯过去的决策和对话细节

发现过期信息 → 在 MEMORY.md 中用 replace 动作覆写对应条目。

## 容量约束（重要）

| 文件 | 字符上限 | 典型条目数 |
|---|---|---|
| MEMORY.md | 2,200 chars | 8-15 条 |
| USER.md | 1,375 chars | 5-10 条 |

**满容时必须合并/替换，不能追加。** 这是 Hermes 的硬约束。