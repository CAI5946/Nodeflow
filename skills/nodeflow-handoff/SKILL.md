---
name: nodeflow-handoff
description: "Use at the end of 1-to-N work to summarize changes, verification, unresolved items, risks, blockers, and next steps."
---

# Nodeflow Handoff

## Node Metadata

- Node ID: `nodeflow-handoff`
- 作用: 汇总完成内容、验证证据、未完成项和风险
- 输入: 原始请求、完成内容、修改文件、验证结果、review 结果、风险
- 输出: 最终回复、可选 `docs/CHANGELOG.md`, `docs/USAGE.md`, `docs/reviews/...`
- 前置条件: 执行、review 或验证已有结论
- Gate: 关键失败未说明或验证缺失时不做完成交接
- 默认 checkpoint: Y，仅 High 风险或发布前需要
- 可跳过条件: orchestrator 需要继续执行后续节点
- 下游节点: none
- 适用风险: Low / Medium / High

## Node Status

- Status: core
- Role: handoff node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Be concise.
- Do not overstate completion.
- Mention verification evidence.
- Mention failures, blockers, and residual risks.
- Respect `document_budget`: do not invent review/report files during handoff.
- Separate completed work from recommendations.

## Inputs

- original request
- completed work
- changed files
- verification result
- review result if present
- unresolved items
- known risks
- `docs/.nodeflow/<nodeflow_id>/state.json`（如存在，用于更新交接状态）

## Process

1. Summarize what changed.
2. List important files when useful.
3. Report verification command and result.
4. Call out unfinished work or blockers.
5. State next step only if it directly follows from the request.
6. Update `docs/CHANGELOG.md`, `docs/USAGE.md`, `docs/reviews/`, or `docs/reports/` only when the nodeflow or user requires it.
7. For Medium/High risk work, include document handling when task-level documents were created or intentionally skipped under `document_budget`.
8. If `docs/.nodeflow/<nodeflow_id>/state.json` exists，将 nodeflow 状态标记为 `completed`，更新 `last_heartbeat` 和 `updated_at`。不覆盖 individual task 的 `failed`/`blocked` 状态（保留失败审计信息）。清空 `blockers` 中已 resolved 的条目。在输出中提示状态文件路径，供下次 session 恢复使用。

## Document Handling & Project Closeout

As part of project closeout, handle research and development documentation carefully to prevent workspace bloat:

- **Durable Facts Consolidation**:
  - Automatically extract any long-lived engineering truths, API contracts, usage instructions, or architectural decisions from temporary plans/specs.
  - Sync and consolidate these facts into permanent project documentation, such as `docs/DECISIONS.md`, `docs/CHANGELOG.md`, or a user manual (`docs/USAGE.md`).
- **Temporary Document Cleanup**:
  - Identify temporary planning (`docs/plans/`) or review (`docs/reviews/`) documents.
  - Recommend moving them to an archive subdirectory (e.g. `docs/plans/archive/` or `docs/reviews/archive/`) to keep the primary `docs/` clean.
  - Never physically delete or archive these task-level documents silently; always list them in the Handoff document handling section and get explicit user confirmation (by replying `1` or `确认`) in the final handoff message.
  
Default Handoff Document Handling:
- Low risk: no document handling section is needed.
- Medium/High risk: list all temporary files created, specify which ones have had their facts consolidated, and propose their archiving/cleanup for user approval.

## Output Shape

```markdown
## 完成内容

## 验证结果

## 文档处理

## 未完成 / 风险

## 下一步
```

For small tasks, use a short paragraph instead of all headings.

## Stop Conditions

Stop and ask if:

- verification has not run but the user expects completion.
- critical failure is unresolved.
- the user asked to continue execution instead of handoff.
