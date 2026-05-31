---
name: nodeflow
description: "Use as the AI-Led Nodeflow entrypoint to clarify requests, classify work, assess risk, propose a node chain, and wait for approval."
---

# Nodeflow

## Node Metadata

- Node ID: `nodeflow`
- Status: core
- Role: entry / orchestrator
- 输入: 用户请求、项目状态、可用 nodeflow nodes、可选 `AGENTS.md` / `docs/CURRENT.md`
- 输出: 已确认需求、nodeflow 类型、风险等级、document_budget、execution_mode、worktree_policy、推荐节点链、可选候选节点、checkpoint、下一步动作
- Gate: 需求未确认前不生成 nodeflow；nodeflow 未确认前不执行
- 默认 checkpoint: Y
- 下游节点: `nodeflow-brief`, `nodeflow-request-router`, `nodeflow-risk-assessment`, selected nodeflow nodes
- 适用风险: Low / Medium / High

## Node Status

- Status: core
- Role: entry / orchestrator
- Replacement: none

## Core Principle

Nodeflow is a risk-adaptive guardrail, not a fixed track.

- Low risk: keep the path light; let the agent execute after concise confirmation.
- Medium risk: add a plan only when ordering, file scope, or verification benefits from it.
- High risk: require confirmed scope/spec, review, checkpoint when needed, and fresh verification evidence.
- The user should remember one entrypoint: `nodeflow`. Internal nodes stay composable but are not user-facing choices by default.

## Core Rules

- Reply in Chinese.
- Confirm the requirement before proposing a node chain.
- Show one recommended chain; show candidate nodes only when a real choice exists.
- Compose the shortest chain that preserves safety.
- Prefer `core` nodes by default.
- Use `specialized` nodes only when the request clearly matches their domain.
- Use `legacy` nodes only when explicitly invoked or no active chain covers the request.
- Do not invent unavailable nodes.
- Respect each selected node's Gate. If a gate fails, stop and report the blocker.
- Decide `document_budget`, `execution_mode`, and `worktree_policy` before execution.
- Wait for nodeflow approval before execution.
- Do not use process weight to replace engineering judgment.
- Do not claim completion without fresh verification evidence from `nodeflow-verify`.
- Treat Nodeflow as a graph with review/verify loops, not a one-pass linear checklist.
- Keep ordinary capability skills out of the node chain; declare them as `capability_skills` when needed.

## Learn Mode Detection

Before intake, detect explicit learn mode commands:

- "开启学习模式" or "learn mode on": if an active state file exists, set `learn_mode: true`; confirm it is enabled.
- "关闭学习模式" or "learn mode off": if an active state file exists, set `learn_mode: false`; confirm it is disabled.
- Otherwise continue normal intake.

When learn mode is enabled, downstream nodes call `nodeflow-learn` at meaningful decision points. Use `AskHuman`; do not hard-code a specific tool.

## Intake

Use the ultra-low risk path (auto-trigger fast-track) when the request is extremely simple and low risk (e.g. spelling correction, single-line configuration, small documentation fix):

```plain text
clear Ultra-low risk request -> auto-trigger execution (no proposal confirmation needed)
```

Use the light path when the request is clear, Low risk, and not creative/product/architecture/behavior-changing work:

```plain text
clear Low-risk request -> concise requirement confirmation -> nodeflow proposal
```

Use the non-code capability direct path when the request is a pure non-code task or exploratory analysis:

```plain text
non-code request -> direct capability skill activation -> output & handoff
```

Use `nodeflow-brainstorming` when the request is creative, a new feature, architecture or interaction work, behavior-changing, oversized, or under-specified:

```plain text
unclear/creative/feature/architecture/behavior-changing request
-> nodeflow-brainstorming
-> confirmed spec
-> user review
-> nodeflow-plan
```

If the request is ambiguous, ask at most 3 targeted questions and stop. If a safe assumption is enough, state it and ask the user to confirm.

## Non-code Capability Direct Activation

For pure non-code tasks or exploratory tasks (such as `research-decision`, `notion-dev-kit-update`, `prompt-optimization`):
- The orchestrator directly activates the corresponding capability skill (e.g. `reference-analysis`, `notion-page`, `prompt-optimize`, `doc-update`) to execute the task.
- Completely bypass the heavy Nodeflow core nodes (such as plan, execute, review, verify, validate).
- Output the final result directly to `nodeflow-handoff` or the user, with no state files or plan files created.

## Requirement Anchor

Before a nodeflow proposal (except for Ultra-low risk auto-trigger and Non-code Direct Activation), create a concise `requirement_anchor`:

```markdown
### Requirement Anchor

- goal:
- must_have:
  - R1:
- must_not:
  - N1:
- acceptance_criteria:
  - A1:
- output:
```

Rules:

- Generate it from the confirmed requirement or the `nodeflow-brainstorming` summary.
- For extremely small Low-risk work, derive at least one `must_have`.
- On nodeflow approval, write it to state.
- Do not proceed to proposal if the anchor is missing.

Detailed templates live in `references/templates.md`.

## Fast Routing

Classify mode:

```plain text
0-to-1 new project / new product / new repo
1-to-N existing project request
```

Classify request type:

```plain text
ultra-low-change
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

For `research-decision`, `notion-dev-kit-update`, `prompt-optimization`, and pure documentation maintenance without code, route through the **Non-code Capability Direct Activation** path.

Classify risk:

```plain text
Ultra-low
- spelling correction, single-line configuration fix, tiny doc update
- zero behavior/state changes, zero production risk
- auto-triggered execution without formal proposal approval

Low
- docs/copy/style/single-file low-risk work
- no behavior/data impact
- simple verification available

Medium
- multi-file changes
- existing behavior/state changes
- scoped refactor
- verification requires tests or manual flow

High
- new project or new feature
- creative, architecture, or behavior-changing work whose desired outcome is not confirmed
- architecture/data model/storage/migration impact
- unknown root cause bug
- release/build/account/payment/security/privacy risk
- broad shared contract changes
```

If risk is not obvious, route through `nodeflow-risk-assessment`.

## Discipline Decisions

Set:

```plain text
document_budget: none | plan-only | spec+plan | durable-evidence
execution_mode: simple-sequential | planned-sequential | subagent-driven | worktree-subagent
worktree_policy: not-needed | recommended | required | exception:<reason>
```

Defaults:

- Low: `document_budget=none`, `execution_mode=simple-sequential`, `worktree_policy=not-needed`.
- Medium: `document_budget=plan-only` only when planning adds value; otherwise `none`.
- High / creative / behavior-changing: confirmed spec/scope before plan; default `document_budget=spec+plan`.
- Complex or parallel implementation: recommend isolated worktree unless there is a concrete exception.

Detailed document budget, execution mode, node status, and default chains live in `references/node-registry.md`.

## References

Read references only when the task needs that detail:

- `references/node-registry.md`: node list, status, default chains, document budget, execution mode, legacy routing.
- `references/capability-skills.md`: normal specialist skills available inside plan, execute, review, verify, or handoff.
- `references/askhuman-protocol.md`: checkpoint, HITL, fallback question format, AskHuman adapter order.
- `references/templates.md`: requirement confirmation, requirement anchor, proposal, checkpoint templates.
- `references/state-management.md`: state initialization, resume, DAG, checkpoint persistence.
- `references/pressure-scenarios.md`: self-test scenarios for editing nodeflow skills.

Do not copy references into the user response unless requested.

## Node Selection Rules

- New project: start from `nodeflow-brief`.
- Existing project: start from `nodeflow-request-router`.
- Unclear bug: use `nodeflow-bug-diagnose` before planning or fixing.
- Creative, architecture, behavior-changing, oversized, or unclear work: use `nodeflow-brainstorming` before `nodeflow-plan` or implementation.
- Confirmed complex existing-project change: use `nodeflow-risk-assessment`, then `nodeflow-change-spec` if scoped behavior still needs writing down.
- File scope, dependencies, or verification need sequencing: use `nodeflow-plan`.
- Code/doc changes: use `nodeflow-execute`.
- Meaningful local changes: use `nodeflow-review`.
- Completion claim: use `nodeflow-verify`.
- Final closure: use `nodeflow-handoff`.

## Proposal Output

Ultra-low risk (auto-trigger):

```markdown
路由：Ultra-low 极轻量风险，自动触发直通通道（无需确认）。直接执行修改并进行验证。
```

Non-code Capability Direct Activation:

```markdown
路由：纯分析/研究或非代码任务，直接激活对应能力 Skill（无需确认）。直接运行并生成交付物。
```

Low risk:

```markdown
路由：Low 风险，`document_budget=none`，`execution_mode=simple-sequential`，按 `nodeflow-execute -> nodeflow-verify -> nodeflow-handoff` 执行。

请确认是否按这个 nodeflow 执行。回复 `确认` 或 `1` 继续。
```

Medium / High risk:

```markdown
## Nodeflow 判断

- 类型：
- 风险：
- document_budget：
- execution_mode：
- worktree_policy：
- 推荐节点链：
- 候选节点：
- checkpoint：
- 跳过节点：
- 需要确认：

请确认是否按这个 nodeflow 执行。回复 `确认` 或 `1` 继续。
```

## Execution Rules

1. Decide whether the request is ultra-low risk, non-code capability direct, light intake, or needs `nodeflow-brainstorming`.
2. For **Ultra-low risk**: auto-trigger execution and verification directly without showing proposal confirmation or asking for approval. Proceed to output handoff with diff.
3. For **Non-code Direct Activation**: directly invoke the corresponding capability skill to execute and output the result, completely bypassing all core node sequences and proposal confirmation.
4. For other risks, confirm the requirement or reviewed spec.
5. Generate `requirement_anchor`.
6. Identify 0-to-1 vs 1-to-N.
7. Classify type and risk.
8. Decide `document_budget`, `execution_mode`, and `worktree_policy`.
9. Compose one recommended chain from the registry.
10. Add candidate nodes only when optional choice is material.
11. Present the proposal and ask for approval using `AskHuman`.
12. After approval, initialize state when needed and execute nodes in order.
13. If review, verification, or validation fails and the issue is fixable within scope, loop back to `nodeflow-execute`.
14. Stop on failed gate, missing input, checkpoint, non-fixable failure, or retry limit.
15. End with verification evidence and handoff.

State initialization and resume rules live in `references/state-management.md`.

## Red Flags

Stop and rethink if you catch yourself doing any of these:

- "This is small, so no need to confirm what success means."
- "A full spec/plan would be safer for this tiny edit."
- "I will list several full nodeflow alternatives just in case."
- "AskHuman is a real tool, so I can call it directly."
- "The process says to continue, even though the requirement is unclear."
- "Verification can wait until the final message."

## Stop Conditions

Stop and ask if:

- the requirement has not been confirmed (except for Ultra-low risk and Non-code Direct Activation).
- request type cannot be classified.
- no safe node chain can be composed.
- the nodeflow proposal has not been approved (except for Ultra-low risk and Non-code Direct Activation).
- a required node is missing.
- the user asks to skip verification for work that claims completion.
