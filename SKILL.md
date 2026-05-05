---
name: neat-freak
description: >
  会话收尾知识审查 — 审查并同步 Hermes Agent 的知识体系（MEMORY.md、
  USER.md、SOUL.md、项目上下文文件），防止知识腐化，让 Agent 越用越聪明。
  MUST trigger when the user says:
  "整理一下", "审查一下", "存档", "收尾", "同步一下", "梳理一下",
  "/neat", "neat", "clean up memory", "sync memory", "tidy up",
  "这个阶段做完了", "会话结束了", "保存一下",
  or any phrase suggesting a session wrap-up where knowledge needs
  reconciliation. Also trigger when the user reports stale memories,
  conflicting info, or wants a clean handoff to the next session.
  Bare "整理" / "tidy" with prior task context counts — do not under-trigger.
  Adapted for Hermes Agent from the original neat-freak by 数字生命卡兹克.
---

# 洁癖 — 会话收尾知识审查（Hermes 版）

> Original by [数字生命卡兹克](https://github.com/KKKKhazix) · [原版 neat-freak](https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak) · [原版文章](https://mp.weixin.qq.com/s/tg1wd-iN2gWHWhXdY0faeg)
>
> 本版本针对 Hermes Agent 场景适配，核心适配：容量感知（MEMORY.md 2200 字符上限、USER.md 1375 字符上限）、渐进式子目录上下文发现、FTS5 会话搜索审查。

> 你是知识库编辑，不是记录员。记录员只会往后追加，编辑会审查全局、合并重复、修正过期、删除废弃。
> 你的工作：让 Agent 的知识体系始终保持干净、准确、对新会话友好。

## 核心原则

**合并优于追加，删除优于保留。**

在 AI 协作场景里，信息多不是优势，信息准才是。一条过期记忆比没有记忆更糟糕——没有记忆时 Agent 至少知道自己不知道，会问；但过期记忆会让它基于错误前提做事。

Hermes 的 MEMORY.md 和 USER.md 有硬性字符上限，这个原则更加重要——空间有限，每个字符都要有价值。

## Hermes 知识体系

Agent 的知识分布在以下层级：

| 层级 | 文件 | 受众 | 容量 | 职责 | 不同步的代价 |
|------|------|------|------|------|-------------|
| **Agent 人格** | `$HERMES_HOME/SOUL.md` | 跨会话的自己 | 无硬限 | 风格、偏好、红线 | 人格漂移 |
| **长期记忆** | `~/.hermes/memories/MEMORY.md` | 跨会话的自己 | **2,200 chars** | 精炼的关键事实、决策、教训 | 下次会话遗忘历史决策 |
| **用户画像** | `~/.hermes/memories/USER.md` | 跨会话的自己 | **1,375 chars** | 用户偏好、沟通风格、习惯 | 忽略用户偏好 |
| **项目上下文** | `.hermes.md` / `AGENTS.md` / `CLAUDE.md` | 项目中的自己 | 20,000 chars（超限截断） | 项目架构、约定、指令 | 违反项目规范 |
| **子目录上下文** | `<subdir>/AGENTS.md` | 子目录中的自己 | 同上 | 子目录级约定 | 子模块规范不一致 |
| **Skills** | `~/.hermes/skills/<name>/SKILL.md` | 按需加载的自己 | 无硬限 | 程序性知识、工作流 | 能力退化 |

### 上下文文件优先级

同一会话只加载一个项目上下文（first match wins）：

1. `.hermes.md` — 最高优先级
2. `AGENTS.md`
3. `CLAUDE.md` — Claude Code 兼容
4. `.cursorrules` — Cursor IDE 兼容

## 执行流程

### 第一步：盘点现状（强制机械式枚举，不能跳过）

**先做 ls，再做判断。**

1. 读核心记忆文件：
   - `~/.hermes/memories/MEMORY.md` — 长期记忆，统计当前字符数和容量占比
   - `~/.hermes/memories/USER.md` — 用户画像，统计当前字符数和容量占比
2. 读人格文件：`~/.hermes/SOUL.md`
3. 读项目上下文文件（按优先级，找到第一个即停）：
   - `<cwd>/.hermes.md`
   - `<cwd>/AGENTS.md`
   - `<cwd>/CLAUDE.md`
   - `<cwd>/.cursorrules`
4. **子目录扫描**：`find . -name AGENTS.md -o -name .hermes.md -o -name CLAUDE.md` → 列出所有子目录上下文文件
5. 列出 skills：`ls ~/.hermes/skills/`
6. 会话搜索审查：用 session search 搜索本次对话涉及的关键实体，检查历史会话中是否有过期信息
7. 回顾本次对话全部内容

**输出一张文件清单**（内部用），每个文件标「评估过 / 要改 / 不用改」+ 记忆文件标容量占比。**漏一个不行**。

### 第二步：识别变更

**不只看对话增量有什么新事实，要看新事实会波及哪些知识层级。**

常见模式：

| 本次对话发生的事 | 要改的文件 |
|---|---|
| 新增/修改偏好 | `USER.md`（replace 动作） |
| 做出重要决策 | `MEMORY.md`（replace/add 动作） |
| 踩坑/教训 | `MEMORY.md`（精炼结论，replace 腾空间） |
| 项目状态变化 | `MEMORY.md`（replace 项目条目） |
| 工具/技能配置变化 | `MEMORY.md`（如有隐性知识） |
| 人格/风格调整 | `SOUL.md` |
| 新增定期任务 | `MEMORY.md`（如需记住任务存在） |
| 发现有过期信息 | `MEMORY.md`（replace 过期条目） |
| 会话搜索中发现过期信息 | `MEMORY.md`（replace 对应条目） |
| 子目录上下文需要更新 | 对应子目录的 `AGENTS.md` |

完整映射表见 [references/sync-matrix.md](references/sync-matrix.md)。

**关键检查**：

1. **容量检查**：MEMORY.md 和 USER.md 是否接近上限？接近 → 先执行合并/删除腾空间
2. **会话搜索**：历史会话中有没有与当前认知矛盾的信息？有 → replace MEMORY.md 对应条目
3. **子目录上下文一致性**：各子目录的 AGENTS.md 之间有无矛盾？有 → 标记在变更摘要
4. **项目上下文与记忆一致性**：AGENTS.md 说用 PostgreSQL 但 MEMORY.md 还写着 MySQL？→ 以更近更新为准

### 第三步：实际修改

**必须真的改文件，不是描述"我会怎么改"。**

**顺序**：先 `MEMORY.md`（合并/修正长期记忆）→ 再 `USER.md`（更新画像）→ 再项目上下文文件 → 最后 `SOUL.md`（如有调整）。

**编辑原则**：

- **合并优于追加**：新信息是对旧信息的更新，用 replace 改旧条目，不要 add 新条目
- **删除优于保留**：完成的任务、推翻的决策、过期上下文，用 remove 删除
- **精确优于冗长**：一条记忆说清楚一件事，每个字符都要有价值
- **绝对时间**：永远 `2026-05-05`，不写"今天"、"最近"
- **面向新会话的自己**：写的时候想象下个会话的你只看 MEMORY.md 就能理解

**容量感知写入**：

1. 写入前检查当前容量
2. 如空间不足，按优先级删除低价值条目（临时上下文 > 已完成任务 > 可合并的重复条目）
3. 优先用 replace 更新已有条目（不增加条目数），而非 add 新条目
4. 每次写入后确认未超限

**项目上下文文件修改**：

- 直接编辑文件内容（不像 MEMORY.md 需要 memory tool）
- 注意不要超过 20,000 字符（超限会被自动截断）
- 子目录 AGENTS.md 修改时，保持与根目录上下文文件的风格一致

### 第四步：自检清单（必须逐项过一遍）

- [ ] 第一步列出的每个文件，都判断了"不用改"或"已改"
- [ ] MEMORY.md 与项目上下文无矛盾
- [ ] 无相对时间遗留（`grep -E "今天|昨天|刚刚|最近|上周|today|yesterday|recently"` 在修改过的文件中清零）
- [ ] 本次对话的关键信息都已落盘（不在对话上下文里"悬空"）
- [ ] USER.md 与实际偏好一致
- [ ] 会话搜索中发现的过期信息已在 MEMORY.md 中覆写
- [ ] 无重复条目
- [ ] **容量未超限**：MEMORY.md ≤ 2,200 chars，USER.md ≤ 1,375 chars
- [ ] **子目录上下文已扫描**：所有子目录的 AGENTS.md / .hermes.md 已检查
- [ ] **项目上下文与记忆一致**：AGENTS.md 和 MEMORY.md 无矛盾描述

哪条打不了勾，**回去补**。

### 第五步：变更摘要

```
## 🧹 同步完成

### 记忆变更
- replace：xxx（原因）
- add：xxx
- remove：xxx（原因）

### 容量状态
- MEMORY.md：xxxx/2200 chars (xx%)
- USER.md：xxxx/1375 chars (xx%)

### 文档变更
- SOUL.md — xxx
- AGENTS.md — xxx
- 子目录上下文 — xxx

### 会话搜索覆写
- 历史会话中"xxx"已过期，已在 MEMORY.md 中 replace

### 未处理
- xxx（为什么没处理）
```

只列有实际变更的条目。没改的不写。

## 特殊情况

**对话没有产生新事实**：审查现有记忆有没有过期/冲突/相对时间——审查本身就有价值。

**记忆容量已满**：按优先级删除低价值条目后再写入。如果全是高价值条目，合并相近条目腾空间。**绝不能因为"放不下"就不写关键信息**——删掉不太重要的旧信息给新信息让路。

**记忆之间出现无法自动判断的矛盾**：列在「未处理」让用户决定。**这是唯一需要用户介入的情况**，其他都自己拍板。

**发现之前的同步漏了东西**：修掉。不要说"那不是这次对话的事"——你就是这个 Agent 的持续编辑，过去的漏洞也归你管。

**项目有多个子目录上下文文件**：逐个审查，发现矛盾时标记在变更摘要。跨子目录的一致性问题由用户决定。

**SOUL.md 修改**：谨慎操作。SOUL.md 是全局人格，修改影响所有会话和所有项目。只在用户明确要求时修改。

## 定期执行

建议通过 Hermes cron 设置每周自动执行全量审查：

```
hermes cron → "执行 /neat 全量审查：盘点所有知识文件（MEMORY.md、USER.md、SOUL.md、项目上下文、子目录上下文），检查过期/矛盾/相对时间，输出审查报告。如发现需修改项，执行修改并输出变更摘要。"
```

这样即使忘记手动触发，知识体系也不会持续腐化。

## 参考资料

- **[references/sync-matrix.md](references/sync-matrix.md)** — 完整的"变更类型 → 要改哪些文件"映射表 + 容量管理策略
- **[references/agent-paths.md](references/agent-paths.md)** — Hermes Agent 知识文件路径速查