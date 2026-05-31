---
name: nodeflow-plan
description: "Use after a confirmed spec or medium-risk request to create an executable implementation plan before editing files."
---

# Nodeflow Plan

## Node Metadata

- Node ID: `nodeflow-plan`
- 作用: 将 spec 或中风险请求拆成可执行计划
- 输入: 已确认 spec 或路由结论、`AGENTS.md`, 相关代码上下文
- 输出: 可执行计划；仅在 `document_budget` 允许时写入 `docs/plans/YYYY-MM-DD-request-name-plan.md`
- 前置条件: spec 已确认，或请求风险为 Medium 且范围明确
- Gate: plan 缺少文件路径、验证命令或预期输出时停止
- 默认 checkpoint: Y
- 可跳过条件: Low 风险请求可直接执行
- 下游节点: `nodeflow-execute`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: planning node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Plan before executing.
- Do not implement code in this skill.
- Every task must have file scope, dependencies, acceptance criteria, verification command, and expected output.
- Respect `document_budget`: Low risk defaults to no plan file; Medium writes a plan only when it adds execution value; High requires confirmed spec/scope first.
- Do not write placeholders such as `TBD`, `TODO`, `fill in`, `later`, `similar to above`, or vague steps like "add appropriate validation".
- Prefer small tasks that can be verified independently.
- Do not mark tasks parallel if they touch the same core file or share unresolved contracts.
- Choose `execution_mode` and `worktree_policy` inside the plan when work is complex or parallelizable.
- Declare `capability_skills` only when specialist rules are needed. Capability skills are used inside execution/review/verification; they do not become Nodeflow nodes or default-chain steps.
- Prefer TDD when the change is logic-heavy, regression-prone, or has a clear failing behavior. If TDD is not useful, state the alternative verification path.
- Self-review the plan before handoff: check spec coverage（含 requirement_anchor 覆盖度矩阵）、placeholder text, file paths, dependencies, verification commands, and design simplicity.
- Design simplicity check: if the same logic or behavior is being added to 3+ files, ask whether it should be a separate node/wrapper instead of scattered injection. Prefer centralized responsibility over distributed duplication.

## Checkpoint Focus

When this node reaches a checkpoint, the `Human Check` should focus on:

- whether the plan scope matches the approved request or spec
- whether file paths and ownership boundaries are correct
- whether acceptance criteria and verification commands prove the requested behavior
- whether any HITL task needs a human decision before execution
- whether the approach is the simplest viable solution: same logic in 3+ files should be a separate node; unnecessary indirection or abstraction should be flagged
- when the upstream brainstorming checkpoint already confirmed the spec and the plan adds no new design decisions, the plan checkpoint may be combined with the spec review. State that the combined review covers both spec approval and plan confirmation.

## Inputs

- confirmed spec or routing conclusion
- `AGENTS.md`
- relevant code and project docs
- existing tests or commands

## Plan File

Decide the plan output based on the project type:

- **全新项目开发 (0-to-1 New Project)**:
  - 直接在当前项目根目录下生成全局的 **`docs/TASKS.md`**。
  - 不生成局部的 `docs/plans/` 文件。
  - `docs/TASKS.md` 包含所有开发阶段、精细排布的有序 Task 列表（包含 dependencies、acceptance_criteria、verify_commands、AFK/HITL 标签及覆盖度矩阵）。

- **已有项目迭代 (1-to-N Existing Project)**:
  - 默认产出局部的 **`docs/plans/YYYY-MM-DD-request-name-plan.md`** 文件。
  - 创建 `docs/plans/` 目录（若不存在）。不要默许静默覆盖已有文件。

Create a plan only when planning adds value:

- Low risk: skip the plan unless the user asks for one.
- Medium risk: create a plan when more than one file/module may change, ordering matters, or verification has multiple steps.
- High, creative, architecture, or behavior-changing work: require a confirmed spec or equivalent confirmed scope before writing the plan.

The plan (whether `docs/TASKS.md` or `docs/plans/`) is a task-level artifact. Long-term product or architectural conclusions must be synced to main project docs (like PRD or ARCH) instead of living only in the plan file.

## Execution Mode and Worktree Policy

Select one mode:

- `simple-sequential`: do not use this node unless the user explicitly asks for a plan.
- `planned-sequential`: default for Medium/High work with shared files or ordered dependencies.
- `subagent-driven`: independent tasks can run in parallel without file conflicts.
- `worktree-subagent`: complex implementation or parallel tasks should use isolated worktrees.

Worktree defaults:

- Recommend isolated worktree for complex implementation or parallel task groups.
- Allow exceptions for docs-only work, small fixes, and environment-sensitive projects; write the reason as `worktree_policy=exception:<reason>`.

## Plan Structure

```markdown
# Plan: Request Name

## Scope

## Document Budget

## Execution Mode

## Worktree Policy

## Capability Skills

List specialist skills required by the task, or `[]` when none are needed.

## Assumptions

## Tasks

### Task 1: Title

- 任务目标：
- requirement_refs：[R1, R2]（该 task 服务的 requirement_anchor 条目 ID）
- 修改文件路径：
- 新建文件路径：
- 输入：
- 输出：
- 依赖任务：
- 是否可并行：Y/N
- 类型：AFK / HITL
- capability_skills：[] 或 [frontend-design, playwright]
- 验收标准：
- 验证命令：
- 预期输出：

## 覆盖度矩阵

| Requirement | Task(s) | 状态 |
|-------------|---------|------|
| R1: <条目> | T1, T3 | ✅ 已覆盖 |
| R2: <条目> | T2 | ✅ 已覆盖 |
| N1: <条目> | 无 | ✅ 未触及 |

## Checkpoints

## Risks
```

## Process

1. Read the spec or routing conclusion.
2. Read `requirement_anchor` from state file（如有）。如果没有 state file（Low-risk 无 state 初始化），从 nodeflow proposal 中的 requirement_anchor 摘要读取。如果都没有，从需求确认文本中提取 must_have/must_not 生成简版 anchor 并写入 plan 头部。
3. Inspect only files needed to identify scope.
4. Define task dependencies.
5. Define exact file paths.
6. Add verification commands from `AGENTS.md` or project config.
7. Mark AFK/HITL.
8. Select `execution_mode` and `worktree_policy`.
9. Select `capability_skills` from `../nodeflow/references/capability-skills.md` only when the task domain needs specialist rules. Do not add a capability skill as a Nodeflow node.
10. **为每个 task 声明 `requirement_refs`**：列出该 task 服务的 requirement_anchor 条目 ID（如 ["R1", "R2"]）。如果一个 task 不服务于任何 must_have 条目，说明它属于基础设施或支撑工作，标注为 `requirement_refs: ["infra"]`。
11. Run plan self-review and fix gaps inline.
    - 形式检查：spec 覆盖、placeholder、文件路径、依赖、验证命令。
    - **覆盖度检查**：生成覆盖度矩阵，验证每个 `must_have` 条目至少被一个 task 的 `requirement_refs` 覆盖，每个 `must_not` 条目不被任何 task 触及。如果有遗漏，补充 task；如果有违规，修改 task 范围。
    - 设计简洁性检查：同一逻辑是否被注入到 3+ 个文件？如果是，应改为独立节点/wrapper 集中维护。方案是否是满足需求的最简方案？是否存在不必要的间接层或抽象？
    - 如果 state file 中 `learn_mode: true`，调用 `nodeflow-learn`（传入：plan 中的技术选型决策、execution mode 选择理由）。
12. Ask for confirmation for Medium/High risk plans.

## Gate

```plain text
Do not execute if the plan lacks file paths, verification commands, expected outputs, acceptance criteria, contains placeholders, has design simplicity issues (same logic in 3+ files without justification), or has requirement_anchor coverage gaps（must_have 条目无 task 覆盖或 must_not 条目被 task 触及）。
```

## Red Flags

Stop and revise the plan if you catch yourself thinking:

- "The plan is detailed enough even though it has no verification command."
- "Parallel work is fine without file ownership."
- "TDD is unnecessary because the fix is obvious" for a regression-prone logic change.
- "This task exists because it feels useful, not because it maps to a requirement."
- "A large plan can skip dependency ordering."

## Stop Conditions

Stop and ask if:

- spec is required but missing.
- file scope is unclear.
- verification command cannot be determined.
- requested parallel work conflicts on core files.
- the plan would require inventing product behavior not present in the spec or request.
- the task is Low risk and the plan would only repeat the user's request without adding execution value.
