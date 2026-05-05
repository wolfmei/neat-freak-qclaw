# 变更影响矩阵

遇到不确定"这次改动要同步哪些文件"时查这张表。

## 对话内容变更 → 文件变更

| 本次对话发生的事 | 要改的文件 |
|---|---|
| 新增/修改用户偏好 | `USER.md` + `self-improving/memory.md` |
| 做出重要决策 | `MEMORY.md`（决策+理由）+ `memory/today.md`（决策过程） |
| 踩坑/教训 | `self-improving/corrections.md`（详细过程）+ `MEMORY.md`（精炼结论） |
| 项目状态变化（启动/暂停/完成） | `MEMORY.md`（项目条目更新）+ `memory/today.md` |
| 工具/技能配置变化 | `TOOLS.md` + `MEMORY.md`（如有隐性知识） |
| 人格/风格调整 | `SOUL.md` + `IDENTITY.md` |
| 新增定期任务（cron） | `MEMORY.md`（任务条目）+ `memory/today.md` |
| 发现有过期信息 | `MEMORY.md`（修正或删除）+ 检查 daily notes |
| LCM 中有过期摘要 | `MEMORY.md`（覆写条目：⚠️ LLM 中 xxx 已过期，实际为 yyy） |
| 发现行为规则 | `self-improving/memory.md`（HOT 规则）+ `MEMORY.md`（如影响全局认知） |
| 跨项目/跨 workspace 改动 | 涉及的所有 workspace 的 MEMORY.md + daily notes |

## 记忆层变更

| 情况 | 处理方式 |
|---|---|
| 过期事实 | 改 MEMORY.md 条目，删除旧内容 |
| 相对时间（"今天"、"最近"） | 全部转绝对日期（`2026-05-05` 而非"今天"） |
| 重复记录（多条说同一件事） | 合并为一条 |
| 已完成的待办 | 删除——知识库不是历史档案 |
| 推翻的决策 | 删除旧条目，留新决策 |
| 跨会话只用一次的临时上下文 | 删除 |
| LCM 中的过期摘要 | MEMORY.md 中写覆写条目，不可编辑 LCM 本身 |

## self-improving 与 Workspace 知识体系的交叉规则

### 什么放 self-improving，什么放 MEMORY.md

| 内容类型 | 放 self-improving | 放 MEMORY.md | 原因 |
|---|---|---|---|
| 行为规则（"总是做 X"） | ✅ memory.md (HOT) | 仅概要 | memory.md 是规则引擎的正式存储 |
| 用户偏好（"我喜欢 X"） | ✅ memory.md (HOT) | ✅ USER.md | memory.md 存可执行规则，USER.md 存人物画像 |
| 纠错记录（"参数名写错了"） | ✅ corrections.md | 仅精炼教训 | corrections 记过程细节，MEMORY 记一句话结论 |
| 事实/决策（"项目 X 暂停了"） | ❌ | ✅ MEMORY.md | 事实不是规则，归 MEMORY.md |
| 项目状态（"LLM-WIKI 已部署"） | ❌ | ✅ MEMORY.md | 项目概览归 MEMORY.md |
| 项目/领域级模式 | ✅ projects/ 或 domains/ | ❌ | 按需加载，不污染全局 |

### 去重判断

当同一条信息在两套系统中都有时：

1. **规则/偏好** → 保留在 `self-improving/memory.md`，MEMORY.md 中删除或简化为一行引用（"见 self-improving/memory.md"）
2. **教训** → `corrections.md` 保留完整过程，`MEMORY.md` 保留一句话结论，两边措辞确保一致
3. **事实** → 保留在 `MEMORY.md`，self-improving 中删除

### 维护检查项

| 检查项 | 条件 | 处理 |
|---|---|---|
| memory.md 超 100 行 | 行数 > 100 | 30天未用模式降频到 projects/ 或 domains/ |
| corrections.md 超 50 条 | 条目 > 50 | 归档最旧的条目 |
| index.md 统计不准 | 行数与实际不符 | 更新行数和 Last Updated |
| 重复条目 | 同一信息两处都有 | 按上表去重 |

## task-summary 文件生命周期

| 年龄 | 处理 |
|---|---|
| < 7天 | 保留，近期可回溯 |
| 7-30天 | 关键信息蒸馏到 MEMORY.md，原文件移入 `memory/archive/` |
| > 30天 | 确认已蒸馏后删除 |

## LCM 审查策略

LCM（Lossless Context Management）的 compacted summary 不可直接编辑，但可能包含过期信息。

**审查方法**：
1. 用 `lcm_grep` 搜索本次对话涉及的关键实体名
2. 检查返回的摘要内容是否与当前认知一致
3. 不一致 → MEMORY.md 中写覆写条目

**覆写条目格式**：
```
- ⚠️ LCM 覆写：[实体名] — LLM 摘要中描述为"旧描述"，实际为"新描述"（更新于 YYYY-MM-DD）
```

**批量覆写**：当 LCM 中某项目的描述已全面过期，写一条总覆写：
```
- ⚠️ LCM 覆写：[项目名] — LLM 中关于此项目的摘要已全面过期，以本 MEMORY.md 为准
```

## Daily Notes 整理原则

- `memory/YYYY-MM-DD.md` 是原始日志，**不在里面删改**历史条目
- 如发现历史 daily note 中有过期信息，在当天 note 末尾追加更正，不回改原文
- 蒸馏到 MEMORY.md 时，优先从 daily notes 中提取"学到了什么"，而非"做了什么"
- daily notes 超过 90 天且已蒸馏的，可移入 `memory/archive/`
