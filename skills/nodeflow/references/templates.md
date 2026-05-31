# Nodeflow Templates

Use these templates only when the active node needs them. Keep the live response
shorter when the task is Low risk.

## Requirement Confirmation

```markdown
## 需求确认

- 我理解的需求：
- 目标产物：
- 必须满足：
- 不做：
- 约束 / 风险：
- 需要补充：

请确认需求是否正确。回复 `确认` 或 `1` 后，我再生成 nodeflow；不会直接执行。
```

## Requirement Anchor

```markdown
### Requirement Anchor

- goal：<一句话目标>
- must_have：
  - R1: <条目>
- must_not：
  - N1: <条目>
- acceptance_criteria：
  - A1: <条目>
- output：<目标产物>
```

Rules:

- `goal` comes from the confirmed requirement.
- `must_have` uses `R1`, `R2`, ...
- `must_not` uses `N1`, `N2`, ...
- `acceptance_criteria` uses `A1`, `A2`, ...
- If the request is very small, derive at least one `must_have`.

## Medium / High Proposal

```markdown
## Nodeflow 判断

- 类型：
- 风险：
- document_budget：
- execution_mode：
- worktree_policy：
- 推荐节点链：
  1. ...
  2. ...
- 候选节点：
  - ...
- checkpoint：
- 跳过节点：
- 需要确认：

请确认是否按这个 nodeflow 执行。回复 `确认` 或 `1` 继续。
```

## Low-Risk Proposal

```markdown
路由：Low 风险，`document_budget=none`，`execution_mode=simple-sequential`，按 `nodeflow-execute -> nodeflow-verify -> nodeflow-handoff` 执行。

请确认是否按这个 nodeflow 执行。回复 `确认` 或 `1` 继续。
```

## Candidate Nodes

Use candidates as optional additions around the recommended chain, not separate
alternative nodeflows.

```markdown
- 候选节点：
  - `nodeflow-change-spec`：如果行为边界仍不清楚，先写 scoped spec；否则跳过。
  - `nodeflow-review`：如果改动超过单文件或影响行为，执行后加入 review。
```
