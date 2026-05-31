---
name: nodeflow-refactor
description: "Use for scoped refactors that preserve behavior, such as cleanup, simplification, naming, file splitting, or duplication reduction."
---

# Nodeflow Refactor

## Node Metadata

- Node ID: `nodeflow-refactor`
- 作用: 在不改变行为的前提下做小范围重构
- 输入: 重构目标、范围、相关代码、`AGENTS.md`
- 输出: 重构计划、代码变更、验证结果
- 前置条件: 用户请求清理、简化、拆分或重组代码
- Gate: 重构计划未确认前不改代码
- 默认 checkpoint: Y
- 可跳过条件: 请求包含行为变化，应路由到 feature 或 bugfix 节点
- 下游节点: `nodeflow-plan`, `nodeflow-execute`, `nodeflow-review`, `nodeflow-verify`
- 适用风险: Medium

## Node Status

- Status: specialized
- Role: refactor node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Behavior must remain unchanged unless the user explicitly approves a behavior change.
- Keep scope limited to the named module, files, or component.
- Do not introduce new architecture, dependencies, or broad rewrites.
- Prefer existing project structure and patterns.
- Do not edit before the user confirms the refactor plan.
- Preserve user changes and ignore unrelated dirty files.
- When reporting a discovered issue during assessment, include a reproduction flow when it can be reproduced by user action, API call, data state, or command.
- After refactoring, verification must include concrete manual steps; for app/user-facing changes, include real-device test steps.

## Confirmation Shorthand

- When waiting for confirmation, treat `1` as confirmation to continue with the current pending step.
- When asking the user to choose from numbered options, treat `1` as selecting option 1 instead.

## Template Prefill Rule

- If the user only invokes this skill, provide an empty template.
- If the user invokes this skill with an initial idea, conservatively prefill only fields clearly supported by the user's text.
- Leave uncertain fields empty. Do not write `待补充`.
- Do not infer hidden behavior constraints, file paths, or refactor goals.
- Ask the user to confirm or complete the template before entering Phase 1.

## Phase 0: Initial Invocation

If details are missing, provide this template and ask the user to confirm or complete it:

```markdown
请按下面模板补充重构信息：

1. 重构范围：
2. 当前问题：
3. 重构目标：
4. 不允许改变的行为：
5. 对应文件：
6. 验收标准：
```

## Phase 1: Assessment

Read the named files and output:

```markdown
## 现状理解

## 主要问题

## 复现流程

## 不应改变的行为

## 可重构边界

等待你确认范围后再给方案。
```

Do not modify code.

## Phase 2: Plan

After scope confirmation, provide:

```markdown
## 问题 / 目标理解

## 修改必要性

## 最佳方案

## 需要修改的文件

## 不会改动的内容

## 风险与影响

## 验证方式

等待你确认后再修改代码。
```

Do not modify code.

## Phase 3: Execute And Verify

Only edit after explicit approval.

After editing, output:

```markdown
## 修改文件

## 重构内容

## 行为是否变化

## 验证结果

## 手动验证路径

## 实机测试步骤

## 最终结论

建议：
```

For Flutter projects, include whether to use hot reload `r`, hot restart `R`, or rerun `flutter run`.

## Stop Conditions

Stop if the refactor reveals a bug, requires new architecture, touches unrelated modules, or changes behavior unexpectedly.
