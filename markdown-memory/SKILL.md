---
name: markdown-memory
description: Distill conversations into layered Markdown memory files (preferences, identity, decisions, insights, learned, project memory). Use when the user says 记住、记一下、整理记忆、更新记忆、同步记忆, when a durable preference/decision/task/insight/identity fact appears, or when a session should load or write the Markdown memory palace.
---

# Markdown 记忆

把对话蒸馏成可分层加载的 Markdown。禁止存原文。完整协议：https://github.com/w1ndys/prompts/blob/main/docs/loci-markdown-generation.md

先回答人，再检查信号。无信号不写。有信号才路由写入。确认最多一句人话，不报路径。

## 何时加载本 skill

- 用户说记住 / 记一下 / 整理记忆 / 更新记忆 / 同步记忆
- 本轮出现可存信号：新偏好、稳定身份、真正决策、明确任务、可复用教训、项目接手状态
- 会话开始需要读 L1，或结束需要整理

闲聊、重复已知事实、一次性情绪、过程噪音：不要加载，不要写。

## 大脑位置

1. 系统提示词或 `AGENTS.md` 里写了大脑根目录 → 用它。
2. 否则找同时满足的目录：存在 `me/preferences.md` 和 `plan.md`。
3. 找不到 → 用中文问用户路径，等回答。不要猜，不要在当前代码仓里新建人生档案。

严肃项目的记忆在该仓库 `.loci/`（或项目约定目录），不要把项目全文拷进大脑。

## 会话开始（L1）

只读这些，不要预加载其它文件：

- `plan.md`
- `me/preferences.md`
- `projects/index.md`
- 当前任务快照（如 `tasks/active.md`）
- 若在已连接项目内：该项目 `.loci/memory.md`

同一会话里，未变化的 L1 不要每轮重读。聊到再读 L2。用户明确问历史才读 L3。

## 写入算法

对每个信号：

1. 未来的 AI 知道这件事会不会明显更有用？否 → 丢弃。
2. 按路由表落到**唯一**文件。允许的双写只有：决策全文 + L1 一行摘要；人卡 + 关系边。
3. 蒸馏成可独立读的结论，带原因。禁止对话转写、会议纪要腔。
4. 当前文件就地改并保持短；历史只追加；决策一个文件。
5. 关键句后加 `<!-- source: conversation @ISO8601 -->`。
6. 更新该文件 `updated`。
7. 在 `activity/YYYY-MM.md` 或 `.loci/activity/YYYY-MM.md` 追加：`- HH:MM · 类别 · 一句人话`
8. 若是决策，问会不会改变当前行为。会 → 只把最小结论上浮到 `plan.md` / 任务快照 / `projects/index.md` / 项目 `memory.md`。不会 → 不上浮。
9. 主回答末尾最多一句确认。新偏好必须在**同一轮回复里已经遵守**。

撤销：用户说「撤销 / undo」→ 回滚本会话最近一次保存（默认同一次改的全部文件）。

## 路由

| 类型 | 落点 | 写法 |
|---|---|---|
| 怎么跟我说话 | `me/preferences.md` | 祈使句，L1，就地改 |
| 我是谁 | `me/identity.md` | 稳定事实，就地改 |
| 已稳定原则 | `me/values.md` | 禁止当天感想直接写入 |
| 精力/睡眠规律 | `me/wellbeing.md` | 规律，不是病历 |
| 新鲜反思 | `me/insights.md` | 背景/洞察/为什么重要/暂定影响/状态=observing |
| 可复用教训 | `me/learned.md` | 追加，带反例 |
| 身份或原则变了 | 先改当前文件，再追加 `me/evolution.md` | 旧/新/原因 |
| 人生方向 | `plan.md` | 就地改进度 |
| 拿走项目仍重要的选择 | 大脑 `decisions/YYYY-MM-DD-slug.md` | 背景/选项/决定/后续 |
| 项目内选择 | 项目 `.loci/decisions/` | 同上，不复制进大脑 |
| 明确待办 | 任务源 | 带时间也不自动当日程 |
| 开会/课/出行 | 日程 | 不要自动变任务 |
| 没想清楚 | `inbox.md` | 一行；不要加载进 L1 |
| 还没认真做的项目 | `projects/side.md` | 短条目 |
| 认真项目 | 项目 `.loci/`；大脑 `projects/index.md` 只一行 | 禁止拷全文 |
| 外部文章/链接 | `references/` | 指针 + `use-for`，不吞全文 |
| 自己的笔记 | `notes/index.md` 一行 | 正文留原处 |
| 人 | `people/<name>.md` | 两个人同时出现必须写关系边 |
| 过期 | `archive/` | 搬家不删除 |

决策测试：把这个项目拿走后，这个决定还重要吗？否 → 项目决策（默认）。是 → 大脑。不确定 → 留项目，不要双写全文。

inbox vs side：念头进 inbox；潜在项目进 side。

调研不是决策。有取舍再写决策，并引用调研。

## 决策模板

文件名 `YYYY-MM-DD-slug.md`。标题就是结论。至少两个真实选项。没选的路必须留下。

```markdown
---
date: YYYY-MM-DD
tags: []
status: active
---

# Decision: <结论>

## Background
## Options
## Decision
### Reasoning
### Trade-offs Accepted
## Follow-up
- [ ]
```

## 洞察模板

```markdown
## YYYY-MM-DD — 短标题

**背景**：
**洞察**：
**为什么重要**：
**暂定影响**：
**状态**：observing
```

未稳定前不要升级到 `values.md`。

## 项目 memory.md

只保留接手上下文，通常 < 150 行：Goal、Current State、Now/Next（1–3 项就地覆盖）、Recent Progress（3–7 条）、仍生效的决策链接、风险。完整流水追加到 `.loci/progress/YYYY-MM.md`。稳定属性进 `.loci/profile.md`。

大脑索引只在新连接项目，或出现对仓库之外也有意义的 insight/milestone 时更新一行。`[private]` 永远不上浮。

项目还没认真起来：放 `projects/side.md`。认真起来后在对话结尾问一次要不要建项目记忆；被拒后不再问。

## 确认话术

好：「记住了：明天下午 3 点看材料。」
坏：「已蒸馏到 `decisions/...`。」「已写入 L1。」

无信号则完全不提记忆。

## GitHub 同步

仅当用户要求同步，或系统提示词明确「写完记忆后推送」时：

1. 在大脑仓库 `git pull --rebase`（或 `git pull`）
2. 只暂存记忆文件，不要夹带无关改动
3. 中文提交：`docs(memory): <一句话>`
4. `git push`

不要把密钥、精确财务、隐私联系方式推进公开仓库。大脑必须是私有仓库。

## 禁止

- 把聊天记录、思考过程、整段对话写进记忆
- 一个巨大 `memory.md` 无限追加
- 当前文件堆历史
- 无选项的「决策」
- 当天感想进 `values.md`
- 项目全文复制进大脑
- 对人回复里暴露路径和内部词
- 承认新偏好但当轮仍用旧称呼
- 无信号也每天空总结
- 用删除代替归档
