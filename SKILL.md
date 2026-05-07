---
name: neat-freak-qclaw
description: >
  会话收尾知识审查 — 审查并同步 Agent 的知识体系（MEMORY.md、daily notes、
  AGENTS.md、USER.md、task-summary、LCM summaries、self-improving、.learnings/），
  防止知识腐化，让 Agent 越用越聪明。MUST trigger when the user says:
  "整理一下", "审查一下", "存档", "收尾", "同步一下", "梳理一下",
  "/neat", "neat", "clean up memory", "sync memory", "tidy up",
  "这个阶段做完了", "会话结束了", "保存一下",
  or any phrase suggesting a session wrap-up where knowledge needs
  reconciliation. Also trigger when the user reports stale memories,
  conflicting info, or wants a clean handoff to the next session.
  Bare "整理" / "tidy" with prior task context counts — do not under-trigger.
  Adapted for QClaw / OpenClaw from the original neat-freak by 数字生命卡兹克.
---

# 洁癖 — 会话收尾知识审查

> Original by [数字生命卡兹克](https://github.com/KKKKhazix) · [原版 neat-freak](https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak) · [原版文章](https://mp.weixin.qq.com/s/tg1wd-iN2gWHWhXdY0faeg)
>
> 本版本针对 QClaw / OpenClaw 个人 Agent 场景深度适配，新增 LCM 审查、self-improving 交叉检查、.learnings/ 经验日志审查、task-summary 生命周期管理。

> 你是知识库编辑，不是记录员。记录员只会往后追加，编辑会审查全局、合并重复、修正过期、删除废弃。
> 你的工作：让 Agent 的知识体系始终保持干净、准确、对新会话友好。

## 核心原则

**合并优于追加，删除优于保留。**

在 AI 协作场景里，信息多不是优势，信息准才是。一条过期记忆比没有记忆更糟糕——没有记忆时 Agent 至少知道自己不知道，会问；但过期记忆会让它基于错误前提做事。

## QClaw 知识体系

Agent 的知识分布在三套并行系统中，neat-freak 要统一审查：

> ⚠️ **防混规则**：
> - `MEMORY.md`（workspace 根目录）和 `~/self-improving/memory.md` 是两个完全不同的文件。前者是你的长期记忆，后者是 self-improving 的 HOT 规则层。
> - `.learnings/` 是 SkillHub 版 self-improving-agent 的经验日志，和 `~/self-improving/` 是两套独立系统。
> 执行任何修改前，必须先确认目标文件的完整路径，绝不能只看文件名就动手。

### 系统一：Workspace 知识体系

| 层级 | 文件 | 受众 | 职责 | 不同步的代价 |
|------|------|------|------|-------------|
| **Agent 人格** | `SOUL.md` + `IDENTITY.md` | 跨会话的自己 | 风格、偏好、红线 | 人格漂移 |
| **长期记忆** | `MEMORY.md` | 跨会话的自己 | 精炼的关键事实、决策、偏好、教训 | 下次会话遗忘历史决策 |
| **工作日志** | `memory/YYYY-MM-DD.md` | 可回溯的自己 | 原始事件记录 | 丢失细节 |

### 系统二：self-improving 记忆体系

| 层级 | 文件 | 受众 | 职责 | 不同步的代价 |
|------|------|------|------|-------------|
| **HOT 规则** | `~/self-improving/memory.md` | 跨会话的自己 | 已确认的规则和偏好（≤100行） | 重复犯错 |
| **纠正日志** | `~/self-improving/corrections.md` | 回溯的自己 | 犯错+修正记录 | 同一个坑踩两次 |
| **WARM 知识** | `~/self-improving/projects/` + `domains/` | 按需加载的自己 | 项目/领域级模式 | 领域知识丢失 |
| **COLD 归档** | `~/self-improving/archive/` | 按需查询的自己 | 久未使用的模式 | 无 |

### 系统三：.learnings/ 经验日志（SkillHub self-improving-agent）

| 层级 | 文件 | 受众 | 职责 | 不同步的代价 |
|------|------|------|------|-------------|
| **经验日志** | `.learnings/LEARNINGS.md` | 跨会话的自己 | 纠正、洞察、知识缺口、最佳实践（结构化 ID） | 重复犯同样的错 |
| **错误日志** | `.learnings/ERRORS.md` | 回溯的自己 | 命令失败、异常、集成错误 | 同一个坑踩两次 |
| **需求日志** | `.learnings/FEATURE_REQUESTS.md` | 规划的自己 | 用户请求的能力 | 遗忘用户需求 |

> `.learnings/` 位于 workspace 根目录下，是 [SkillHub self-improving-agent](https://clawhub.ai/self-improving-agent) 的存储目录。与 `~/self-improving/` 是完全独立的两个系统，不要混淆。

### 三套系统的分工

| 内容类型 | 放哪里 | 为什么 |
|---|---|---|
| 事实、决策、项目状态 | `MEMORY.md` | 全局概览，新会话第一个读 |
| 教训（犯过错+怎么改） | `corrections.md` + `MEMORY.md` | corrections 记细节过程，MEMORY 记精炼结论 |
| 已确认的行为规则 | `self-improving/memory.md` | 这是它的本职，HOT 层自动加载 |
| 偏好（用户说"我总是..."） | `self-improving/memory.md` + `USER.md` | memory.md 存可执行规则，USER.md 存人物画像 |
| 项目/领域级模式 | `~/self-improving/projects/` + `domains/` | 按需加载，不污染全局 |
| 开发经验/错误/需求 | `.learnings/LEARNINGS.md` + `ERRORS.md` + `FEATURE_REQUESTS.md` | 结构化追踪，带生命周期管理 |

**去重原则**：同一条信息在多套系统中都有时，保留在职责更匹配的那一边，另一边删掉或改为交叉引用。三套系统之间尤其注意：
- `~/self-improving/corrections.md` vs `.learnings/LEARNINGS.md`：功能重叠，corrections 偏行为纠错，LEARNINGS 偏开发经验，按内容性质分流
- `~/self-improving/memory.md` vs `.learnings/LEARNINGS.md`：memory.md 存已晋升的精炼规则，LEARNINGS.md 存待处理的原始经验

## 执行流程

### 第一步：盘点现状（强制机械式枚举，不能跳过）

**先做 ls，再做判断。**

1. 读 workspace 根目录下所有核心文件（**读取时用完整绝对路径，不要简写**）：
   - `<workspace>/MEMORY.md` — **不是** `~/self-improving/memory.md`
   - `SOUL.md`、`IDENTITY.md`
   - `AGENTS.md`、`USER.md`、`TOOLS.md`
   - `HEARTBEAT.md`（如有）
2. `ls memory/` → 列出所有 daily notes
3. 读今天和昨天的 `memory/YYYY-MM-DD.md`
4. 扫描 task-summary 文件：`ls task-summary_*.md 2>/dev/null`
5. LCM 审查：`lcm_grep` 搜索本次对话涉及的关键实体，检查 compacted summary 中是否有过期信息
6. **self-improving 审查**（**所有路径以 `~/self-improving/` 开头，不要与 workspace 的 `memory/` 混淆**）：
   - 检测 `~/self-improving/` 是否存在
   - 存在 → 读 `~/self-improving/memory.md`（HOT 规则）、`~/self-improving/corrections.md`、`~/self-improving/index.md`
   - `ls ~/self-improving/projects/` 和 `~/self-improving/domains/`
   - 检查 `index.md` 行数统计是否与实际一致
7. **.learnings/ 审查**（**路径为 `<workspace>/.learnings/`，不要与 `~/self-improving/` 混淆**）：
   - 检测 `<workspace>/.learnings/` 是否存在
   - 存在 → 读 `.learnings/LEARNINGS.md`、`.learnings/ERRORS.md`、`.learnings/FEATURE_REQUESTS.md`
   - 统计各文件中 pending/in_progress/resolved/promoted 的数量
   - 找 Recurrence-Count ≥ 3 的条目（应晋升但未晋升）
   - 找 status=pending 但根据对话上下文实际已解决的条目
8. 回顾本次对话全部内容

**输出一张文件清单**（内部用），每个文件标「评估过 / 要改 / 不用改」。**漏一个不行**。

### 第二步：识别变更

**不只看对话增量有什么新事实，要看新事实会波及哪些知识层级。**

常见模式：

| 本次对话发生的事 | 要改的文件 |
|---|---|
| 新增/修改偏好 | `USER.md` + `self-improving/memory.md` |
| 做出重要决策 | `MEMORY.md`（决策+理由）+ `memory/today.md`（过程） |
| 踩坑/教训 | `self-improving/corrections.md`（详细过程）+ `MEMORY.md`（精炼结论） |
| 项目状态变化 | `MEMORY.md`（项目条目更新）+ `memory/today.md` |
| 工具/技能配置变化 | `TOOLS.md` + `MEMORY.md`（如有隐性知识） |
| 人格/风格调整 | `SOUL.md` + `IDENTITY.md` |
| 新增定期任务（cron） | `MEMORY.md`（任务条目）+ `memory/today.md` |
| 发现有过期信息 | `MEMORY.md`（修正或删除）+ 检查 daily notes |
| LCM 中有过期摘要 | `MEMORY.md`（覆写条目：⚠️ LLM 中 xxx 已过期，实际为 yyy） |
| 跨项目/跨 workspace 改动 | 涉及的所有 workspace 的 MEMORY.md + daily notes |

完整映射表见 [references/sync-matrix.md](references/sync-matrix.md)。

**关键检查**：

1. **LCM**：compacted summary 里有没有与当前认知矛盾的信息？有 → 在 MEMORY.md 中写覆写条目
2. **self-improving 交叉检查**：
   - `memory.md` 的 HOT 规则与 `MEMORY.md` 的教训条目有无重复？有 → 保留在更合适的一边，另一边删除或简化
   - `corrections.md` 中已 FIXED 的条目，其精炼结论是否已存在于 `MEMORY.md`？没有 → 补上
   - `index.md` 的行数统计是否准确？不准确 → 更新
   - `memory.md` 是否超过 100 行限制？超过 → 需要降频（demote WARM）
3. **.learnings/ 交叉检查**：
   - `.learnings/LEARNINGS.md` 中 status=pending 的条目，是否在本次对话中实际已解决？→ 标记为 resolved
   - 是否有 Recurrence-Count ≥ 3 但 status 仍为 pending 的条目？→ 应晋升到 CLAUDE.md / AGENTS.md / TOOLS.md / SOUL.md
   - `.learnings/LEARNINGS.md` 与 `~/self-improving/corrections.md` 是否有重复记录？→ 保留在更合适的一边，另一边加 See Also
   - `.learnings/ERRORS.md` 中 status=pending 的错误，当前是否已修复？→ 标记为 resolved
4. **三系统一致性**：同一条信息在多处描述不同 → 以更近更新为准，同步其他处

### 第三步：实际修改

**必须真的改文件，不是描述"我会怎么改"。**

**顺序**：先 `<workspace>/memory/today.md`（追加当日记录）→ 再 `<workspace>/MEMORY.md`（合并/修正长期记忆）→ 再 `~/self-improving/`（如存在）→ 再 `.learnings/`（如存在）→ 最后检查其他文件。

> ⚠️ **修改前必做路径确认**：写文件前，在内部清单中明确标注完整路径。`MEMORY.md` = workspace 长期记忆；`~/self-improving/memory.md` = self-improving HOT 规则。搞混了就是 bug。

**编辑原则**：

- **合并优于追加**：新信息是对旧信息的更新，改旧条目，不要再加一条
- **删除优于保留**：完成的临时计划、推翻的决策、过期的上下文，删掉
- **精确优于冗长**：一条记忆说清楚一件事
- **绝对时间**：永远 `2026-05-05`，不写"今天"、"最近"
- **面向新会话的自己**：写的时候想象下个会话的你只看 MEMORY.md 就能理解

**task-summary 文件处理**：

| 年龄 | 处理 |
|---|---|
| < 7天 | 保留 |
| 7-30天 | 关键信息蒸馏到 MEMORY.md，原文件移入 `memory/archive/` |
| > 30天 | 确认已蒸馏后删除 |

**self-improving 维护**（如已安装）：

- `memory.md` 超 100 行 → 按 self-improving 的降频规则（30天未用→WARM）处理
- `corrections.md` 超 50 条 → 归档最旧的条目
- `index.md` 统计不准确 → 更新行数
- 重复条目 → 保留在职责更匹配的系统，另一边删除或交叉引用

**.learnings/ 维护**（如已安装）：

- status=pending 但已解决 → 标记为 resolved，添加 Resolution 块
- Recurrence-Count ≥ 3 且跨 2+ 任务 → 晋升到对应的 workspace 文件（CLAUDE.md / AGENTS.md / TOOLS.md / SOUL.md），原条目标记为 promoted
- 重复条目（同 Pattern-Key 或相似 Summary）→ 合并为一条，更新 Recurrence-Count 和 See Also
- `.learnings/ERRORS.md` 中超过 30 天的 pending 条目 → 确认是否仍相关，不相关标 wont_fix
- 与 `~/self-improving/corrections.md` 重复 → 保留在更合适的一边，另一边加交叉引用

### 第四步：自检清单（必须逐项过一遍）

- [ ] 第一步列出的每个文件，都判断了"不用改"或"已改"
- [ ] MEMORY.md 与 daily notes 无矛盾
- [ ] 无相对时间遗留（`grep -E "今天|昨天|刚刚|最近|上周|today|yesterday|recently"` 在修改过的文件中清零）
- [ ] 本次对话的关键信息都已落盘（不在对话上下文里"悬空"）
- [ ] USER.md 与实际偏好一致
- [ ] LCM 中发现的过期信息已在 MEMORY.md 中覆写
- [ ] task-summary 文件已按年龄策略处理
- [ ] 无重复条目
- [ ] **self-improving 已检测**：存在则已审查，不存在则标记"未安装，跳过"
- [ ] **.learnings/ 已检测**：存在则已审查，不存在则标记"未安装，跳过"
- [ ] **三系统无矛盾**：MEMORY.md、self-improving/memory.md、.learnings/LEARNINGS.md 无冲突描述
- [ ] **self-improving 索引准确**：index.md 行数与实际一致
- [ ] **.learnings/ 无积压**：无长期 pending 未处理条目，应晋升的已晋升

哪条打不了勾，**回去补**。

### 第五步：变更摘要

```
## 🧹 同步完成

### 记忆变更
- 更新：xxx（原因）
- 新增：xxx
- 删除：xxx（原因）

### 文档变更
- MEMORY.md — xxx
- memory/2026-05-05.md — xxx
- USER.md — xxx

### self-improving 变更
- memory.md — xxx
- corrections.md — xxx
- index.md — 行数更新
- 去重：xxx（保留在 xxx，从 xxx 删除）

### .learnings/ 变更
- LEARNINGS.md — resolved: x 条, promoted: x 条, 合并: x 条
- ERRORS.md — resolved: x 条, wont_fix: x 条
- FEATURE_REQUESTS.md — xxx
- 晋升目标：xxx → AGENTS.md / TOOLS.md / SOUL.md
- 与 self-improving 去重：xxx

### LCM 覆写
- ⚠️ LCM 摘要中"xxx"已过期，已在 MEMORY.md 中覆写为"yyy"

### task-summary 清理
- 归档：x 个文件（7-30天）
- 删除：x 个文件（>30天，已蒸馏）

### 未处理
- xxx（为什么没处理）
```

只列有实际变更的条目。没改的不写。

## 特殊情况

**对话没有产生新事实**：审查现有记忆有没有过期/冲突/相对时间——审查本身就有价值。

**记忆之间出现无法自动判断的矛盾**：列在「未处理」让用户决定。**这是唯一需要用户介入的情况**，其他都自己拍板。

**发现之前的同步漏了东西**：修掉。不要说"那不是这次对话的事"——你就是这个 Agent 的持续编辑，过去的漏洞也归你管。

**LCM 中有大量过期信息**：逐条覆写成本太高时，在 MEMORY.md 中写一条总覆写（"LCM 中关于 xxx 项目的描述已全面过期，以本文档为准"），而非逐条罗列。

**self-improving 未安装**：在变更摘要中标记"self-improving 未安装，跳过"，不报错。

**.learnings/ 未安装**：在变更摘要中标记".learnings/ 未安装，跳过"，不报错。

**self-improving 与 MEMORY.md 有重复**：判断职责归属——规则/偏好归 self-improving，事实/决策归 MEMORY.md。两边的描述都保留时确保措辞一致。

**.learnings/ 与 self-improving 有重复**：`corrections.md` 偏行为纠错规则，`LEARNINGS.md` 偏开发经验日志。同一条经验在两边都有时，判断归属：
- 行为规则/偏好 → 保留在 self-improving，LEARNINGS.md 中标 See Also
- 开发经验/技术坑 → 保留在 LEARNINGS.md，corrections.md 中标交叉引用

## 定期执行

建议通过 cron 设置每周日 09:00 自动执行全量审查：

```
cron → isolated agentTurn → "执行 /neat 全量审查：盘点 workspace 所有知识文件，检测 ~/self-improving/ 和 .learnings/ 是否存在并审查，检查过期/矛盾/相对时间，输出审查报告。如发现需修改项，执行修改并输出变更摘要。"
```

这样即使忘记手动触发，知识体系也不会持续腐化。

## 参考资料

- **[references/sync-matrix.md](references/sync-matrix.md)** — 完整的"变更类型 → 要改哪些文件"映射表
- **[references/agent-paths.md](references/agent-paths.md)** — QClaw / OpenClaw 知识文件路径速查
