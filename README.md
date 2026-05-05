# neat-freak — 会话收尾知识审查（Hermes 版）

> 🌿 `hermes` 分支 — 适配 [Hermes Agent](https://github.com/nousresearch/hermes-agent)
>
> 🌿 `main` 分支 — 适配 QClaw / OpenClaw

会话结束后，审查并同步 Agent 的知识体系，防止知识腐化。

**核心原则：合并优于追加，删除优于保留。**

过期信息比没有信息更危险——Agent 会基于错误前提自信地做决策。

## 这解决什么问题？

每次和 Agent 对话，会产生新的决策、偏好、教训。这些信息散落在：
- `MEMORY.md` — 长期记忆（2200 字符上限）
- `USER.md` — 用户画像（1375 字符上限）
- `SOUL.md` — Agent 人格
- 项目上下文（`.hermes.md` / `AGENTS.md` / `CLAUDE.md`）
- 子目录上下文

如果不主动审查同步：
- 过期信息留在记忆里 → Agent 基于错误前提决策
- 重复信息堆积 → 容量浪费
- 项目上下文与记忆矛盾 → 行为不一致

## 安装

```bash
# 克隆仓库
git clone -b hermes https://github.com/wolfmei/neat-freak-qclaw.git

# 复制到 Hermes skills 目录
cp -r neat-freak-qclaw ~/.hermes/skills/neat-freak
```

## 使用

在对话中触发：

```
/neat
"整理一下"
"存档"
"收尾"
```

或设置每周自动审查（通过 Hermes cron）。

## 5 步流程

1. **盘点现状** — 机械式枚举所有知识文件，统计容量占比
2. **识别变更** — 本次对话的增量信息影响哪些文件
3. **实际修改** — 按序修改，容量感知写入
4. **自检清单** — 逐项确认，漏了回去补
5. **变更摘要** — 输出改了什么、容量状态

## Hermes 适配要点

| 特性 | 说明 |
|------|------|
| **容量感知** | MEMORY.md 2200 字符、USER.md 1375 字符硬性上限，写入前必须检查容量 |
| **渐进式子目录** | 自动扫描项目所有子目录的 AGENTS.md，检查跨目录一致性 |
| **FTS5 会话搜索** | 替代 LCM 审查，搜索历史会话中的过期信息 |
| **memory tool** | 通过 add/replace/remove 动作修改记忆，而非直接编辑文件 |
| **上下文优先级** | .hermes.md > AGENTS.md > CLAUDE.md > .cursorrules |

## 示例场景

### HomeDash 项目收尾

刚完成 HomeDash 项目的 webhook 重构：

```
/neat
```

Agent 执行审查后发现：
- MEMORY.md 中 HomeDash 条目还是"轮询模式" → replace 为"webhook 模式"
- USER.md 中无通信偏好 → add "用户偏好 webhook 而非轮询"
- 会话搜索发现之前的讨论中提到"轮询不可靠" → 已在 MEMORY.md 中体现
- 子目录 `frontend/AGENTS.md` 中 API 端点描述需要更新 → 直接编辑

输出变更摘要 + 容量状态。

## 致谢

- [数字生命卡兹克](https://github.com/KKKKhazix) — 原版 [neat-freak](https://github.com/KKKKhazix/khazix-skills/tree/main/neat-freak) 及 [洁癖.Skill 文章](https://mp.weixin.qq.com/s/tg1wd-iN2gWHWhXdY0faeg)

## License

MIT