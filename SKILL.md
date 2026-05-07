---
name: neat-freak-qclaw
description: >
  会话收尾知识审查 — 审查并同步 Agent 的知识体系（MEMORY.md、daily notes、
  AGENTS.md、USER.md、task-summary、LCM summaries、self-improving、.learnings/），
  防止知识腐化，让 Agent 越用越聪明。MUST trigger when the user says:
  "审查一下", "知识审查", "记忆审查", "存档", "收尾", "同步知识",
  "梳理记忆", "/neat", "neat", "clean up memory", "sync memory",
  "知识存档", "这个阶段做完了", "会话结束了", "保存记忆",
  "neat freak", "洁癖",
  or any phrase suggesting a session wrap-up where knowledge needs
  reconciliation. Also trigger when the user reports stale memories,
  conflicting info, or wants a clean handoff to the next session.
  Do NOT trigger on "整理" alone — that routes to file-skill (file/desktop organization).
  Adapted for QClaw / OpenClaw from the original neat-freak by 数字生命卡兹克.
---

# 洁癖 — 会话收尾知识审查

> Original by [数字生命卡兹克](https://github.com/KKKKhazix) · [原版 neat-freak](https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak)

**核心原则：合并优于追加，删除优于保留。** 你是知识库编辑，不是记录员。

## ⚠️ 防混规则（每次执行前必读）

| 易混文件 | 区别 |
|---------|------|
| `<workspace>/MEMORY.md` | 长期记忆（事实、决策、教训） |
| `~/self-improving/memory.md` | HOT 规则层（行为规则、偏好，≤100行） |
| `<workspace>/.learnings/` | SkillHub 经验日志，与 `~/self-improving/` 完全独立 |

**修改前必须确认完整路径，不能只看文件名。** 详细路径见 [references/agent-paths.md](references/agent-paths.md)。

## 执行流程

### 第一步：盘点现状

**先 ls，再判断。** 按顺序读取：

1. **workspace 核心文件**：`MEMORY.md`、`SOUL.md`、`IDENTITY.md`、`AGENTS.md`、`USER.md`、`TOOLS.md`、`HEARTBEAT.md`
2. **daily notes**：`ls memory/`，读今天和昨天的
3. **task-summary**：`ls task-summary_*.md`
4. **LCM**：`lcm_grep` 搜索本次对话关键实体，检查摘要是否过期
5. **self-improving**（如 `~/self-improving/` 存在）：读 `memory.md`、`corrections.md`、`index.md`；`ls projects/` `domains/`
6. **.learnings/**（如 `<workspace>/.learnings/` 存在）：读 `LEARNINGS.md`、`ERRORS.md`、`FEATURE_REQUESTS.md`；找 Recurrence-Count ≥ 3 的条目
7. 回顾本次对话全部内容

输出内部文件清单，每个标「评估过 / 要改 / 不用改」。**漏一个不行。**

### 第二步：识别变更

查 [references/sync-matrix.md](references/sync-matrix.md) 确定"对话发生了什么 → 要改哪些文件"。

**关键检查**（逐项过，不能跳）：

1. **LCM**：`lcm_grep` 搜索本次对话关键实体，compacted summary 与当前认知矛盾？→ MEMORY.md 写覆写条目
2. **self-improving 交叉**：
   - `memory.md` HOT 规则 vs `MEMORY.md` 教训条目 → 重复则保留更合适的一边
   - `corrections.md` 已 FIXED 条目 → 精炼结论是否已在 `MEMORY.md`？没有→补上
   - `index.md` 行数 → 与实际一致？不一致→更新
   - `memory.md` 超 100 行 → 按降频规则处理
3. **.learnings/ 交叉**：
   - `LEARNINGS.md` status=pending → 对话中已解决？→ 标 resolved
   - Recurrence-Count ≥ 3 → 应晋升到 workspace 文件
   - 与 `corrections.md` 重复 → 保留更合适的一边
4. **三系统一致性**：同一信息多处描述不同 → 以更近更新为准

去重判断规则详见 [references/sync-matrix.md](references/sync-matrix.md)。

### 第三步：实际修改

**必须真的改文件，不是描述"我会怎么改"。**

**顺序**：`memory/today.md`（追加）→ `MEMORY.md`（合并/修正）→ `~/self-improving/`（如存在）→ `.learnings/`（如存在）→ 其他文件

**编辑原则**：
- 合并优于追加（更新旧条目，不加新条目）
- 删除优于保留（完成的计划、推翻的决策、过期上下文 → 删）
- 绝对时间（`2026-05-05`，不写"今天""最近"）
- 面向新会话的自己（只看 MEMORY.md 就能理解）

**⚠️ 删除确认规则**：
- 删除整条记忆、修改超过 3 行、修改 SOUL.md/IDENTITY.md → 变更摘要中标注 ⚠️，等用户确认后再执行
- 新增、合并、小修（≤3 行）→ 自己拍板，直接执行
- 用户说"全部执行"/"直接改" → 跳过确认，全部执行

**task-summary**：< 7天保留 | 7-30天蒸馏到 MEMORY.md + 移入 `memory/archive/` | > 30天确认已蒸馏后删除

**self-improving 维护**：memory.md 超 100 行→降频 | corrections.md 超 50 条→归档 | index.md 不准→更新

**.learnings/ 维护**：pending 已解决→resolved | Recurrence-Count ≥ 3→晋升 | 超 30 天 pending→确认是否 wont_fix

### 第四步：自检清单

- [ ] 第一步每个文件都判断了"不用改"或"已改"
- [ ] MEMORY.md 与 daily notes 无矛盾
- [ ] 无相对时间（`grep -E "今天|昨天|刚刚|最近|上周|today|yesterday|recently"` 在修改文件中清零）
- [ ] 关键信息已落盘（不在上下文里"悬空"）
- [ ] USER.md 与实际偏好一致
- [ ] LCM 过期信息已在 MEMORY.md 覆写
- [ ] task-summary 已按年龄策略处理
- [ ] 无重复条目
- [ ] self-improving 已检测（存在则已审查，不存在则标记跳过）
- [ ] .learnings/ 已检测（存在则已审查，不存在则标记跳过）
- [ ] 三系统无矛盾（MEMORY.md / self-improving/memory.md / .learnings/LEARNINGS.md）
- [ ] .learnings/ 无积压（应晋升的已晋升）

**哪条打不了勾，回去补。**

### 第五步：变更摘要

输出修改汇总。**有 ⚠️ 标记的删除/大改项，等用户确认后再执行**。格式：

```
## 🧹 同步完成

### 记忆变更
- 更新/新增/删除：xxx（原因）

### self-improving 变更 / .learnings/ 变更 / LCM 覆写 / task-summary 清理
- （只列有实际变更的，没改不写）

### ⚠️ 待确认
- [ ] 删除：xxx（原因） — 等用户确认

### 未处理
- xxx（为什么没处理）
```

无待确认项时省略 ⚠️ 节。用户说"直接改"则无需等确认。

## 特殊情况

| 情况 | 处理 |
|------|------|
| 对话无新事实 | 审查存量过期/冲突/相对时间——审查本身有价值 |
| 记忆矛盾无法自动判断 | 列在「未处理」让用户决定 |
| 发现之前的同步遗漏 | 修掉，不管是不是本次对话的 |
| LCM 大量过期 | MEMORY.md 写总覆写（"LCM 中关于 xxx 已全面过期，以本文档为准"） |
| self-improving / .learnings/ 未安装 | 标记"未安装，跳过"，不报错 |
| 两系统有重复 | 按职责分流：规则→self-improving，事实→MEMORY.md，开发经验→.learnings/ |

## 参考资料

- **[references/sync-matrix.md](references/sync-matrix.md)** — 变更类型→文件映射、去重判断、LCM 策略、task-summary 生命周期
- **[references/agent-paths.md](references/agent-paths.md)** — 知识文件路径速查（含 Windows 适配）
