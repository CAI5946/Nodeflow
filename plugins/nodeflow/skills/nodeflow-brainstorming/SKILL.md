---
name: nodeflow-brainstorming
description: "Use before nodeflow planning for creative, feature, architecture, behavior-changing, oversized, or unclear requests that need a hard design/spec gate."
---

# Nodeflow Brainstorming

## Node Metadata

- Node ID: `nodeflow-brainstorming`
- 作用: 在 spec / plan / implementation 前澄清创意、产品、架构、交互或行为变化需求
- 输入: 用户原始请求、项目文档、相关代码/文件、既有决策、约束和风险线索
- 输出: 已确认的需求总结、方案比较、Decision Log、设计 spec、自审结果、用户 review 结论
- 前置条件: 请求涉及创意、新功能、架构、交互、行为变化，或需求/验收标准不清
- Gate: 设计和书面 spec 未经用户确认前，不进入 `nodeflow-plan` 或实现
- 默认 checkpoint: Y
- 可跳过条件: Low 风险且需求、范围、验收和验证方式都清楚
- 下游节点: `nodeflow-plan`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: brainstorming / design gate
- Replacement: none

## Core Rules

- Do not write implementation code, scaffold projects, edit behavior, or call implementation nodes while this node is active.
- Use the smallest process that still locks the design. Do not force full specs onto obvious Low-risk work.
- Explore context before asking detailed questions.
- Ask one focused question at a time when a question is needed.
- Prefer multiple-choice questions when they reduce ambiguity.
- For oversized requests, stop early and split into sub-project specs before refining details.
- Keep a running Decision Log.
- Present 2-3 approaches only when there are real alternatives.
- Lead with the recommended approach and explain tradeoffs.
- Write the final spec to `docs/specs/YYYY-MM-DD-request-name.md` when a durable spec is needed.
- Do not commit the spec unless the user explicitly asks.
- After spec self-review, require user review before moving to `nodeflow-plan`.

## Process

1. Explore the current project context:
   - read relevant project docs, files, recent decisions, and existing nodeflow artifacts.
   - separate what already exists from what is proposed.
   - identify implicit constraints and unconfirmed assumptions.
2. Check scope:
   - if the request contains multiple independent systems, decompose it into sub-projects.
   - choose the first sub-project for this brainstorming cycle.
3. Clarify intent:
   - ask one question per message only when the answer affects the design.
   - clarify purpose, target users, constraints, success criteria, non-goals, and acceptance criteria.
4. Clarify non-functional requirements:
   - performance expectations.
   - scale: users, data, traffic, or project size.
   - security and privacy constraints.
   - reliability or availability needs.
   - maintenance and ownership expectations.
   - if the user is unsure, propose explicit assumptions.
5. Lock understanding:
   - provide an Understanding Summary.
   - list assumptions and open questions.
   - ask the user to confirm or correct the summary.
   - do not proceed to approach comparison until confirmed.
6. Compare approaches:
   - propose 2-3 viable approaches when applicable.
   - include complexity, extensibility, risk, maintenance, and verification tradeoffs.
   - recommend one approach.
   - 如果 state file 中 `learn_mode: true`，调用 `nodeflow-learn`（传入：当前决策点描述、推荐方案、备选方案）。
7. Present the design incrementally:
   - keep each section proportional to complexity.
   - cover architecture, components, data flow, state, edge cases, error handling, and testing where relevant.
   - pause for confirmation after meaningful sections.
8. Write the spec:
   - create `docs/specs/YYYY-MM-DD-request-name.md` only after the design is validated.
   - do not overwrite an existing spec silently.
9. Self-review the spec:
   - scan for placeholders.
   - fix contradictions.
   - remove ambiguity.
   - confirm scope is small enough for one plan.
10. Ask the user to review the written spec.
11. After user approval, hand off to `nodeflow-plan`.

## Understanding Summary Gate

Before approach comparison, output:

```markdown
## 需求总结

- 要做什么：
- 为什么做：
- 给谁用：
- 关键约束：
- 明确非目标：
- 验收标准：

## 假设

- 

## 未决问题

- 

请确认或修正以上内容；确认后再进入方案设计。
```

用户确认后，立即生成 `requirement_anchor` 结构（写入 state file 的 `requirement_anchor` 字段，或暂存供后续写入）：

```markdown
### Requirement Anchor

- goal：<一句话目标>
- must_have：
  - R1: <条目>
  - R2: <条目>
- must_not：
  - N1: <条目>
- acceptance_criteria：
  - A1: <条目>
  - A2: <条目>
- output：<目标产物>
```

规则：
- 从"要做什么"提取 `goal`
- 从"必须满足"和验收标准提取 `must_have`，每条分配 R1/R2... ID
- 从"明确非目标"提取 `must_not`，每条分配 N1/N2... ID
- 从"验收标准"提取 `acceptance_criteria`，每条分配 A1/A2... ID
- 这些 ID 在后续 plan（task 的 `requirement_refs`）和 review（逐条对标）中使用

## Decision Log

Maintain decisions in this shape:

```markdown
## Decision Log

| Decision | Alternatives | Rationale |
| --- | --- | --- |
|  |  |  |
```

Record only decisions that affect scope, behavior, architecture, UX, data, validation, or tradeoffs.

## Spec File

Default path:

```plain text
docs/specs/YYYY-MM-DD-request-name.md
```

Use this structure:

```markdown
# Spec: Request Name

## 背景

## 需求总结

## 目标

## 非目标

## 非功能需求

## 方案比较

## 推荐方案

## 设计

## Decision Log

## 风险

## 验收标准

## 需要用户 Review
```

If a spec decision becomes lasting project truth, also propose syncing it into the appropriate main doc such as `docs/PRD.md`, `docs/ARCH.md`, `docs/DECISIONS.md`, or `docs/NODEFLOW.md`.

## Spec Self-Review

Before asking for user review, check:

- Placeholder scan: no `TBD`, `TODO`, empty required sections, or vague placeholders.
- Internal consistency: goals, non-goals, design, risks, and acceptance criteria do not contradict each other.
- Scope check: the spec is small enough for one `nodeflow-plan`; otherwise split it.
- Ambiguity check: requirements with multiple interpretations are resolved or explicitly listed for review.
- Path check: spec is under `docs/specs/`, not `docs/superpowers/specs/`.
- Git boundary check: no commit unless explicitly requested.

Fix issues inline before user review.

## Handoff To Plan

After user review approval, provide:

```markdown
## Brainstorming Handoff

- Spec:
- Requirement Anchor:
  - goal:
  - must_have: [R1, R2, ...]
  - must_not: [N1, N2, ...]
  - acceptance_criteria: [A1, A2, ...]
- Approved approach:
- Key decisions:
- Known risks:
- Required verification:
- Next node: `nodeflow-plan`
```

如果已有 state file，在 handoff 前将 `requirement_anchor` 写入 state file。

## Stop Conditions

Stop and ask if:

- the real goal cannot be stated clearly.
- acceptance criteria cannot be defined.
- the user has not confirmed the Understanding Summary.
- the user has not approved the written spec.
- the request is too broad for one spec and must be decomposed.
- the user asks to skip this gate for High-risk, creative, architecture, or behavior-changing work.
