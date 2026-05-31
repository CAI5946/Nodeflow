---
name: nodeflow-risk-assessment
description: "Use before modifying an existing project when risk, impact scope, spec need, plan need, checkpoint, or verification is unclear."
---

# Nodeflow Risk Assessment

## Node Metadata

- Node ID: `nodeflow-risk-assessment`
- 作用: 评估修改影响范围、潜在风险和验证要求
- 输入: 用户请求、路由结论、相关代码/文档上下文、`AGENTS.md`
- 输出: 风险等级、影响范围、是否需要 spec、是否需要 plan、document_budget、execution_mode、worktree_policy、是否需要 checkpoint、建议节点链、验证要求
- 前置条件: 请求涉及修改、重构、修复、发布或可能影响现有行为
- Gate: 风险等级和影响范围不清时不进入执行
- 默认 checkpoint: Y，仅 Medium / High 风险需要
- 可跳过条件: 纯说明/调研/Prompt 优化，或 Low 风险且影响范围显然单一
- 下游节点: `nodeflow-change-spec`, `nodeflow-plan`, `nodeflow-execute`, `nodeflow-review`, `nodeflow-verify`
- 适用风险: Low / Medium / High

## Node Status

- Status: core
- Role: risk node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Assess before modifying.
- Focus on blast radius, not implementation details.
- Do not edit code in this node.
- Do not overstate risk; classify based on concrete impact.
- Own the decisions for spec, plan, checkpoint, and verification requirements.
- Own the decisions for `document_budget`, `execution_mode`, and `worktree_policy`.
- Treat `nodeflow-brainstorming` as mandatory when desired behavior, creative direction, architecture, or acceptance criteria are not confirmed.
- Keep `nodeflow-request-router` responsible only for request type and initial route.
- If risk cannot be assessed from available context, ask for the smallest missing input.

## Inputs

- user request
- routing conclusion from `nodeflow-request-router` or `nodeflow`
- relevant files or docs
- `AGENTS.md`
- current git diff when available

## Risk Dimensions

Check:

- affected files and modules
- creative/product/architecture decisions still open
- user-visible behavior
- data model, storage, migration, or persistence impact
- API, interface, or shared contract changes
- state management or lifecycle impact
- platform/build/release impact
- test coverage and verification difficulty
- rollback difficulty
- dependency or configuration changes
- security, privacy, account, payment, or production risk

## Risk Classification

```plain text
Low
- single file or docs/style/copy change
- no behavior or data impact
- simple verification available

Medium
- multiple files
- behavior/state changes
- scoped refactor
- existing feature modification
- feature polish with confirmed target behavior
- verification needs tests or manual flow

High
- new feature or architecture change
- creative, product, architecture, or behavior-changing work without confirmed scope/spec
- data model/storage/migration impact
- release/build/signing/account/payment/security/privacy impact
- unknown root cause bug
- broad shared contract changes
```

## Output Shape

```markdown
## 风险评估

- 风险等级：
- 影响范围：
- 可能受影响文件 / 模块：
- 用户可见变化：
- 数据 / 状态影响：
- 潜在回归：
- 验证要求：
- 是否需要 spec：
- 是否需要 plan：
- document_budget：
- execution_mode：
- worktree_policy：
- 是否需要 checkpoint：
- 推荐下一节点：
```

## Routing Guidance

- Low: `nodeflow-execute -> nodeflow-verify -> nodeflow-handoff`
- Medium: `nodeflow-plan -> nodeflow-execute -> nodeflow-review -> nodeflow-verify -> nodeflow-handoff`
- High: `nodeflow-change-spec -> nodeflow-plan -> nodeflow-execute -> nodeflow-review -> nodeflow-verify -> nodeflow-handoff`
- Unknown root cause bug: `nodeflow-bug-diagnose -> nodeflow-plan -> nodeflow-execute -> nodeflow-review -> nodeflow-verify -> nodeflow-handoff`
- Release/build/signing/store risk: `nodeflow-release-prep -> nodeflow-validate -> nodeflow-handoff`

## Decision Rules

- Need spec when behavior, acceptance criteria, user flow, or product decision is not fully defined.
- Need plan when more than one file/module may change, ordering matters, or verification has multiple steps.
- Need checkpoint when risk is Medium/High or user-visible behavior can change materially.
- Skip checkpoint only for Low risk tasks with obvious scope and simple verification.

## Execution Discipline

Set these explicitly:

- `document_budget=none`: Low risk and most review/report results; keep evidence in the reply.
- `document_budget=plan-only`: Medium risk only when ordering, multi-file scope, or multi-step verification benefits from a plan.
- `document_budget=spec+plan`: High, creative, architecture, or behavior-changing work after scope is confirmed.
- `document_budget=durable-evidence`: release, migration, failed checks, security/privacy, production risk, or cross-agent handoff.
- `execution_mode=simple-sequential`: Low risk, single-threaded, no plan needed.
- `execution_mode=planned-sequential`: plan exists but tasks share state or files.
- `execution_mode=subagent-driven`: independent task groups can run concurrently in the same workspace without file conflicts.
- `execution_mode=worktree-subagent`: complex implementation or parallel work benefits from isolated worktrees.
- `worktree_policy=recommended`: default for complex implementation or parallel tasks.
- `worktree_policy=exception:<reason>`: docs-only, small fix, environment-sensitive project, or workspace setup makes worktree unsafe or low ROI.

## Task Document Policy

- Low risk: do not create task-level documents by default; report the result and verification in the final reply.
- Medium risk: create a plan only when ordering, multi-file scope, or multi-step verification matters; default path is `docs/plans/YYYY-MM-DD-request-name-plan.md`.
- High risk, creative, architecture, or behavior-changing work: create or confirm a scoped spec before planning; default path is `docs/specs/YYYY-MM-DD-request-name.md`.
- Review and verification reports are not created by default. Use `docs/reviews/` or `docs/reports/` only when traceability, failed checks, release risk, or handoff requires durable evidence.
- Main facts belong in stable docs such as `docs/PRD.md`, `docs/ARCH.md`, `docs/DECISIONS.md`, and `docs/CHANGELOG.md`; task files must not become the only source of lasting project truth.

## Stop Conditions

Stop and ask if:

- affected area cannot be identified.
- request involves secrets, accounts, payment, release, migration, or production data and context is insufficient.
- user asks to skip risk assessment for High-risk work.
