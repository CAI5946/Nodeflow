---
name: nodeflow-learn
description: "Learn mode node: at nodeflow decision points, generate learning questions for the user, answer targeted, and persist knowledge entries."
---

# Nodeflow Learn

## Node Metadata

- Node ID: `nodeflow-learn`
- 作用: 在 nodeflow 决策点生成引导式学习问题，用户选择后针对性回答，持久化知识条目
- 输入: 当前决策点上下文（节点名、决策描述、备选方案）
- 输出: 知识条目写入 `docs/knowledge/`
- 前置条件: state file 中 `learn_mode: true`
- Gate: 用户未选择问题前不生成知识条目
- 默认 checkpoint: N
- 可跳过条件: learn_mode 为 false，或用户选择跳过
- 下游节点: 返回调用方继续执行
- 适用风险: N/A（横切关注点，非独立阶段）

## Node Status

- Status: specialized
- Role: learn mode handler
- Replacement: none

## Core Rules

- 只在 `learn_mode: true` 时被调用，调用方负责 flag 检测。
- 不打断执行流：用户选择"跳过"时立即返回，不产生任何产物。
- 只记录通用技术知识（type: general），不记录项目特定决策。
- 每次调用最多生成一个知识条目。
- 问题必须针对传入的具体决策，不泛泛。

## Process

1. 接收调用方传入的上下文：
   - `source_node`：调用方节点名（如 nodeflow-brainstorming）
   - `decision`：当前决策点描述（如"选择方案 A 而非方案 B"）
   - `alternatives`：备选方案列表（如有）
2. 根据决策上下文，生成 2-3 个具体的学习问题：
   - 问题围绕"为什么选这个"、"其他方案的原理"、"底层机制"等
   - 不泛泛（好："为什么用 composition 而不是 inheritance？"，差："想了解设计模式吗？"）
3. 通过 `AskHuman` 提问，选项包含 2-3 个学习问题 + "跳过本次"。
   - Claude Code：如 `AskUserQuestion` 可用，优先使用。
   - Codex Plan mode：如 `request_user_input` 可用，优先使用。
   - Codex Default mode 或其他 Agent：回退到纯文本选项，要求用户回复 `1/2/3` 或 `跳过`。
4. 用户选择后：
   - 选择"跳过" → 直接返回，不写入任何文件
   - 选择某个问题 → 进入步骤 5
   - 使用 Other 自由提问 → 进入步骤 5
5. 针对性回答：
   - 解释 why（为什么这样选择）
   - 解释 alternatives（其他方案的优劣）
   - 解释 principles（底层原理）
6. 将问答写入 `docs/knowledge/YYYY-MM-DD-<topic-slug>.md`，格式见 `docs/knowledge/README.md`。
7. 返回调用方继续执行。

## 知识条目格式

路径：`docs/knowledge/YYYY-MM-DD-<topic-slug>.md`

```markdown
---
date: YYYY-MM-DD
type: general
topic: 主题分类
source_node: 调用方节点名
tags: [tag1, tag2]
---

## 问题

用户选择的具体问题

## 回答

针对性解释（why + alternatives + principles）

## 关联

相关知识条目链接（如有）
```

## Stop Conditions

- learn_mode 为 false（不应被调用，但如果被调用则直接返回）
- 用户选择跳过
- 传入的决策上下文为空或不充分（返回并提示调用方）
