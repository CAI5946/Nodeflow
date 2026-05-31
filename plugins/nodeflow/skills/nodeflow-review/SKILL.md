---
name: nodeflow-review
description: "Use after local changes or execution to review request alignment, bugs, regressions, architecture drift, and missing verification."
---

# Nodeflow Review

## Node Metadata

- Node ID: `nodeflow-review`
- 作用: 对改动执行 Spec Review 和 Code Quality Review
- 输入: 原始请求或 spec、plan、diff、验证结果、`AGENTS.md`
- 输出: findings、open questions、verification gaps、summary
- 前置条件: 已有本轮改动或待审查 diff
- Gate: 未解决 P1/P2 问题时不标记完成
- 默认 checkpoint: N
- 可跳过条件: Low 风险无代码改动且验证已覆盖
- 下游节点: `nodeflow-verify`, `nodeflow-execute`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: review node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Findings first.
- Review the actual diff and evidence.
- For complex work, run two stages in order: spec compliance first, code quality second.
- Separate Spec Review from Code Quality Review even when the final answer is concise.
- Do not rewrite code unless the user asks to fix findings.
- If the task declared `capability_skills`, review the result against those specialist rules without treating the capability skill as a Nodeflow node.
- When called from `nodeflow-execute`, operate as one or more review profiles from `nodeflow-execute/references/review-loop.md`.
- If there are no issues, say so and mention residual risk or test gaps.

## Inputs

- original request or spec
- plan if present
- diff
- test or verification results
- declared `capability_skills` if present
- requested review profile, if called as part of a review matrix
- `AGENTS.md`

## Review Layers

For Medium/High, creative, architecture, or behavior-changing work, complete both layers. Do not let code cleanliness compensate for failing the approved spec.

When a specific review profile is requested, perform that profile only and return findings in the same severity format:

- `spec-review-agent`: Spec Review and requirement_anchor coverage.
- `code-quality-review-agent`: Code Quality Review.
- `capability-review-agent`: Capability Skill Review.
- `risk-review-agent`: release/security/privacy/migration/production risk review.
- `verification-review-agent`: verify that the proposed command or observed evidence covers the task; use `nodeflow-verify` for actual command execution.

### Spec Review

Check:

- satisfies original request
- matches confirmed spec or plan
- covers acceptance criteria
- does not implement explicit non-goals
- does not miss key user path
- **逐条对标 requirement_anchor**（如有；从 state file 或 plan 头部读取）：
  - 每个 `must_have` 条目是否被至少一个 task 覆盖
  - 每个 `must_not` 条目是否未被实现
  - 每个 `acceptance_criteria` 条目是否有验证证据
  - 如果有遗漏或违规，标记为 P1 finding
  - 如果无 requirement_anchor（极简 Low-risk 无 anchor），此项跳过

### Code Quality Review

Check:

- bugs or regressions
- duplicate implementation
- architecture drift
- unclear ownership boundaries
- missing tests or verification
- unrelated refactor
- style or convention mismatch

### Feature Quality Checklist

Use only relevant items when the change affects user-facing behavior:

- 功能闭环：输入 -> 处理 -> 存储/同步 -> 输出 -> 反馈是否完整。
- 主路径：用户能进入、完成关键输入、提交后有明确反馈、结果页或列表正确展示。
- 异常路径：失败、中断、重复操作、误操作是否有处理。
- 状态完整性：Loading、Error、Empty、Success、Retry 是否合理；表单检查输入中、校验失败、提交中、防重复提交。
- 状态结构：是否存在多个 bool 混乱、状态命名不一致、状态触发条件不清。
- 数据正确性：保存、更新、删除、页面重进、列表刷新、统计刷新是否一致。
- 本地/同步：本地保存、云同步成功/失败、离线待同步、重试、冲突或删除策略是否清楚。
- 边界输入：空值、超长文本、极端数值、特殊字符、emoji、缺字段数据、重复快速点击。
- 布局交互：小屏、大字体、深色模式、键盘、安全区、点击区域、滑动、返回手势。
- 中断场景：弱网、无网、超时、服务端错误、切后台恢复、页面被系统杀掉后回来。
- 日志：核心保存、编辑、删除、支付、同步等关键动作是否有必要日志，且不泄露敏感数据。

### Capability Skill Review

When `capability_skills` is present, check only the relevant specialist rules.

For `frontend-design`, check:

- responsive layout and stable dimensions
- clear visual hierarchy and scannable UI
- text does not overflow or overlap
- interaction controls match the task and existing design system
- no unrelated decorative or marketing-style UI was added outside scope

## Output Shape

```markdown
## Findings

- [P1/P2/P3] file:line - issue

## Spec / Plan Compliance

- 覆盖原始请求：
- 覆盖验收标准：
- 是否实现非目标：
- 缺失范围：

### Requirement Anchor 对标（如有 requirement_anchor）

| ID | 条目 | 覆盖状态 | 说明 |
|----|------|---------|------|
| R1 | <must_have 条目> | ✅ covered / ⚠️ missing / ❌ violated | |
| N1 | <must_not 条目> | ✅ not touched / ❌ violated | |
| A1 | <acceptance_criteria> | ✅ verified / ⚠️ unverified | |

## Code Quality / Regression Risk

- 潜在 bug：
- 回归风险：
- 重复实现：
- 架构/边界漂移：
- 无关改动：

## Capability Skill Compliance

- 使用的 capability_skills：
- 是否符合对应规则：

## Open Questions

## Verification Gaps

## Summary
```

## Review File Policy

Do not create a review file by default. Review/report results belong in the conversation unless durability is required. Write `docs/reviews/YYYY-MM-DD-request-name-review.md` only when the review must be durable:

- unresolved P1/P2 findings need follow-up
- review is part of release, migration, security, privacy, or other high-risk work
- another agent or human needs the review as handoff evidence
- the user explicitly asks to keep a review record

For normal 1-N work, report review results in the conversation and let `nodeflow-handoff` decide whether document handling needs to mention retained task files.

## Completion Rule

```plain text
Do not mark the task complete if review finds unresolved P1/P2 issues.
```

如果 state file 中 `learn_mode: true` 且 review 涉及值得深入的技术知识点，调用 `nodeflow-learn`（传入：review 发现的技术知识点描述）。

## Red Flags

Stop and sharpen the review if you catch yourself thinking:

- "No tests failed, so there are no issues."
- "The code is clean, so it probably matches the spec."
- "This unrelated refactor is harmless."
- "Verification is missing, but the diff is obvious."
- "I should summarize first and maybe mention findings later."

## Auto-Retry Integration

当 nodeflow-execute 触发自动修复循环时：

1. Review 节点接收修复后的代码
2. 重新执行完整的 review 流程（Spec Review + Code Quality Review）
3. 如果仍有 P1/P2 问题，返回 findings 给 execute 节点
4. 如果无问题，返回通过结果

### 角色定义

- Review 节点**只负责发现问题并报告**，不负责修复
- 修复由 nodeflow-execute 节点负责
- Review 节点需要清晰区分 P1/P2/P3 问题，以便 execute 节点判断是否需要触发修复

### 重试时的 Review 重点

当接收到重试请求时，除了常规检查外，还需关注：

- 上次发现的 P1/P2 问题是否已修复
- 修复是否引入新的问题
- 修复是否在 task scope 内

## Stop Conditions

Stop and ask if:

- original request or expected behavior is unavailable.
- diff is too broad to review safely.
- verification was required but no result is available.
