---
name: nodeflow-request-router
description: "Use to classify existing-project requests and route feature, bugfix, UI, docs, refactor, release, research, or maintenance work."
---

# Nodeflow Request

## Node Metadata

- Node ID: `nodeflow-request-router`
- Status: core
- Role: 1-to-N request classifier
- 输入: 用户请求、`AGENTS.md`、可选 `docs/CURRENT.md`、必要项目上下文
- 输出: 请求类型、初始路由、是否需要 `nodeflow-risk-assessment`、是否需要 `nodeflow-brainstorming`、document_budget、execution_mode
- Gate: 请求类型无法判断时停止
- 默认 checkpoint: N
- 可跳过条件: `nodeflow` 已经明确节点链且用户确认
- 下游节点: `nodeflow-risk-assessment`, `nodeflow-bug-diagnose`, `nodeflow-execute`, `nodeflow-plan`, capability skills
- 适用风险: Low / Medium / High

## Node Status

- Status: core
- Role: 1-to-N request classifier
- Replacement: none

## Core Rules

- Reply in Chinese.
- Classify request type before execution.
- Read only the context needed to classify the request.
- Do not implement code inside this node.
- Do not perform full risk assessment here; use `nodeflow-risk-assessment` when scope, impact, or verification is not obvious.
- For creative work, new features, architecture, behavior changes, oversized requests, or unclear requirements, route to `nodeflow-brainstorming` before execution.
- If request type cannot be judged, ask one focused question.
- Do not let "quick" requests skip verification.

## Request Types

```plain text
small-change
feature
bugfix
ui-polish
documentation
refactor
release-prep
research-decision
notion-dev-kit-update
prompt-optimization
```

## Initial Routing

```plain text
small-change       -> nodeflow-execute -> nodeflow-verify -> nodeflow-handoff
feature            -> nodeflow-brainstorming -> nodeflow-plan
bugfix             -> nodeflow-bug-diagnose
ui-polish          -> nodeflow-risk-assessment or nodeflow-plan (declaring `ui-polish` capability)
documentation      -> nodeflow-execute (declaring `doc-update` capability) -> nodeflow-verify -> nodeflow-handoff
refactor           -> nodeflow-risk-assessment
release-prep       -> nodeflow-release-prep -> nodeflow-validate -> nodeflow-handoff
research-decision  -> use `reference-analysis` capability, then nodeflow-handoff
notion-dev-kit-update -> use `notion-page` capability, then nodeflow-handoff
prompt-optimization -> use `prompt-optimize` capability, then nodeflow-handoff
```

Use `nodeflow-risk-assessment` before execution when:

- affected files/modules are not obvious.
- creative, product, architecture, or behavior-changing decisions are not confirmed.
- behavior, state, data, release, or regression impact is possible.
- more than one file may change.
- verification is not trivial.
- the user asks for a new feature or scoped refactor.

## Routing Discipline

- Low risk defaults to `document_budget=none` and `execution_mode=simple-sequential`.
- Medium risk uses `document_budget=plan-only` only when plan adds execution value; otherwise keep it in conversation.
- High, creative, or behavior-changing work requires confirmed spec/scope before plan.
- Complex implementation or parallelizable work should be considered for `subagent-driven` or `worktree-subagent`; docs-only and small fixes usually stay sequential.

## Output Shape

```markdown
## 路由结论

- 请求类型：
- 初始路径：
- 是否需要 risk-assessment：
- nodeflow-brainstorming gate：
- document_budget：
- execution_mode：
- 下一步：
```

## Stop Conditions

Stop and ask if:

- request type is unclear.
- user goal and constraints conflict.
- the next node cannot be chosen from available nodeflow nodes.
