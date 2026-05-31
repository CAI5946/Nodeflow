---
name: nodeflow-change-spec
description: "Use for complex, ambiguous, high-risk, or behavior-changing existing-project requests that need a confirmed spec before implementation."
---

# Nodeflow Spec

## Node Metadata

- Node ID: `nodeflow-change-spec`
- 作用: 将复杂或高风险需求整理成可确认 spec
- 输入: 用户请求、路由结论、`AGENTS.md`, `docs/CURRENT.md`, 相关上下文
- 输出: `docs/specs/YYYY-MM-DD-request-name.md`
- 前置条件: 请求复杂、模糊、高风险或改变产品行为
- Gate: spec 未确认前不进入 plan 或实现
- 默认 checkpoint: Y
- 可跳过条件: Low 风险请求或已有已确认 spec
- 下游节点: `nodeflow-plan`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: spec node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Produce a spec, not implementation.
- Do not write code.
- Do not design full architecture unless the spec needs a small technical option comparison.
- Ask one focused question at a time when needed.
- Give 2-3 options only when there is a real decision.
- Wait for confirmation before moving to plan or execution for High-risk work.

## Inputs

- user request
- routing conclusion from `nodeflow-request-router`
- `AGENTS.md`
- `docs/CURRENT.md`
- relevant project docs and files

## Spec File

Default output:

```plain text
docs/specs/YYYY-MM-DD-request-name.md
```

Create `docs/specs/` if missing. Do not overwrite an existing spec silently.

Only create a spec when at least one condition applies:

- behavior, acceptance criteria, user flow, or product decision is not fully defined.
- the request is High risk.
- the change affects multiple modules or shared contracts.
- the change may alter data, storage, privacy, release, account, payment, or security behavior.

Do not create a spec for Low-risk copy, docs, style, config, or obvious single-file fixes. If a spec decision becomes lasting project truth, sync it into the appropriate main doc such as `docs/PRD.md`, `docs/ARCH.md`, or `docs/DECISIONS.md`.

## Spec Structure

```markdown
# Spec: Request Name

## 背景

## 目标

## 非目标

## 用户路径 / 使用场景

## 方案比较

## 推荐方案

## 风险

## 验收标准

## 需要确认
```

## Process

1. Read the necessary context.
2. Restate the real user goal.
3. Separate goals from non-goals.
4. Identify the affected user path or system behavior.
5. Compare options when applicable.
6. Recommend one approach with tradeoffs.
7. Define acceptance criteria.
8. Ask for confirmation.

## Gate

```plain text
High-risk or complex work must not proceed to implementation until spec is confirmed.
```

## Stop Conditions

Stop and ask if:

- the goal cannot be stated clearly.
- acceptance criteria cannot be defined.
- product direction needs human choice.
- the user asks to skip spec for high-risk work.
