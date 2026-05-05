# 🧹 neat-freak-qclaw

> 会话收尾知识审查 — 让你的 QClaw Agent 越用越聪明

**Based on [neat-freak](https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak) by [数字生命卡兹克](https://github.com/KKKKhazix)** · 原版文章：[开源「洁癖.skill」，让你的Agent越用越聪明](https://mp.weixin.qq.com/s/tg1wd-iN2gWHWhXdY0faeg)

---

## 这是什么

每次在 Agent 里干完一件事，跑一下 `/neat`，它会：

1. **盘点**所有知识文件（MEMORY.md、daily notes、USER.md、self-improving 等）
2. **识别**本次对话产生了哪些需要落盘的信息
3. **修改**知识文件——合并重复、修正过期、删除废弃
4. **自检**确保没漏改、没矛盾、没相对时间
5. **输出**变更摘要

核心原则：**合并优于追加，删除优于保留。**

过期信息比没有信息更糟糕——没有记忆时 Agent 至少知道自己不知道，会问；但过期记忆会让它基于错误前提做事。

## 与原版的区别

原版 [neat-freak](https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak) 面向**软件开发项目**（代码→文档同步），兼容 Claude Code / Codex / OpenCode / OpenClaw 多平台。

本版本专门适配 **QClaw / OpenClaw 个人 Agent 场景**，核心改动：

| 项目 | 原版 | 本版 |
|---|---|---|
| 知识体系 | 三类：Agent记忆 + CLAUDE.md + docs/ | 两套：Workspace（MEMORY.md/daily notes）+ self-improving |
| 受众 | 自己 + AI同事 + 人类同事 | 未来的自己 |
| 审查对象 | API路由、环境变量、数据库表、集成指南 | 决策、偏好、教训、项目状态 |
| LCM 审查 | ❌ | ✅ 用 lcm_grep 检查 compacted summary 中的过期信息 |
| self-improving 交叉 | ❌ | ✅ 检测安装、去重、一致性、维护（行数限制/索引准确性） |
| task-summary 生命周期 | ❌ | ✅ 7天保留 → 30天归档 → 超期删除 |
| 定期执行 | ❌ | ✅ Cron 每周全量审查 |
| 跨项目文档对齐 | ✅ 重点能力 | ❌ 不适用 |

五步流程框架、核心原则、编辑原则均来自原版。

## 安装

### 方式一：让 Agent 安装

```
帮我安装这个 skill：https://github.com/wolfmei/neat-freak-qclaw
```

### 方式二：手动安装

```bash
# 克隆到 QClaw skills 目录
git clone https://github.com/wolfmei/neat-freak-qclaw.git ~/.qclaw/skills/neat-freak
```

### 方式三：SkillHub

```
skillhub install neat-freak-qclaw
```

## 使用

在对话中说以下任一触发词：

- `/neat`、`/sync`
- "整理一下"、"审查一下"、"同步一下"、"梳理一下"
- "收尾"、"这个阶段做完了"、"会话结束了"
- "clean up memory"、"sync memory"、"tidy up"

### 定期自动执行

建议配置 Cron 每周日 09:00 自动全量审查：

```
cron → isolated agentTurn → "执行 /neat 全量审查"
```

## 示例输出

```
## 🧹 同步完成

### 记忆变更
- 更新：HomeDash 项目状态从"调研中"改为"已上线"（2026-04-12 部署完成）
- 新增：用户偏好——回复用中文，代码注释用英文
- 删除：旧版 API 对接方案（已废弃，改用 webhook）
- 合并：3 条关于 HomeDash 的零散记录 → 1 条完整项目条目

### 文档变更
- MEMORY.md — 项目状态更新 + 合并重复条目 + 覆写 LCM 过期摘要
- memory/2026-04-12.md — 追加当日决策记录（选 webhook 而非轮询的理由）
- USER.md — 新增语言偏好

### self-improving 变更
- memory.md — 与 MEMORY.md 去重："回复用中文"规则保留在 memory.md，MEMORY.md 中删除重复条目
- corrections.md — 新增 1 条：误用 v1 API 端点 → 应使用 v2
- index.md — 行数更新（12→14, 24→26）

### LCM 覆写
- ⚠️ LCM 摘要中"HomeDash 使用轮询方案"已过期，实际已改为 webhook，已在 MEMORY.md 中覆写

### task-summary 清理
- 归档：2 个文件（7-30天）→ memory/archive/
- 删除：1 个文件（>30天，已蒸馏到 MEMORY.md）

### 未处理
- 无
```

## 文件结构

```
neat-freak/
├── SKILL.md                    # 主技能文件
├── references/
│   ├── sync-matrix.md          # 变更类型 → 要改哪些文件 的映射表
│   └── agent-paths.md          # QClaw/OpenClaw 知识文件路径速查
└── scripts/                    # 预留
```

## 致谢

- **数字生命卡兹克** — 原版 [neat-freak](https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak) 作者，五步流程框架和核心原则的提出者
- 原版文章：[开源「洁癖.skill」，让你的Agent越用越聪明](https://mp.weixin.qq.com/s/tg1wd-iN2gWHWhXdY0faeg)

## License

MIT License — 详见 [LICENSE](./LICENSE)
