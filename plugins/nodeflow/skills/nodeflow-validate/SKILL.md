---
name: nodeflow-validate
description: "Use after implementation tasks to run project checks, verify core paths, record failures, and write docs/test-report.md."
---

# Nodeflow Validate

## Node Metadata

- Node ID: `nodeflow-validate`
- 作用: 执行项目级验证并生成测试报告
- 输入: `AGENTS.md`, `docs/TASKS.md`, 项目代码、可选 `docs/CHANGELOG.md`
- 输出: `docs/test-report.md`
- 前置条件: 实现任务已完成或需要项目级验证
- Gate: 验证命令无法确定、关键依赖不可用或关键检查失败时停止
- 默认 checkpoint: N
- 可跳过条件: Low 风险任务已由 `nodeflow-verify` 覆盖且不需要报告
- 下游节点: `nodeflow-project-closeout`, `nodeflow-handoff`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: validation node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Validate observable behavior; do not add new features.
- Use commands from `AGENTS.md` and verification commands from `docs/TASKS.md`.
- Read command output before summarizing results.
- Record failures honestly.
- Do not claim the project is ready if critical checks fail.

## Inputs

Required:

- `AGENTS.md`
- `docs/TASKS.md`
- project code

Recommended if present:

- `docs/CHANGELOG.md`
- `docs/ASSUMPTIONS.md`
- `docs/ARCH.md`
- `docs/CONTEXT.md`

## Actions

1. Read validation commands from `AGENTS.md`.
2. Read task verification evidence from `docs/TASKS.md` and `docs/CHANGELOG.md`.
3. Run the project-level checks that are appropriate for the repo:
   - tests
   - lint/analyze
   - build
   - smoke run
4. Check for:
   - startup failure
   - obvious runtime errors
   - core flow unavailable
   - file structure drift
   - duplicate implementation
   - unresolved assumptions
   - multi-agent interface conflict
5. Generate `docs/test-report.md`.

## test-report.md Structure

```markdown
# Test Report

## 运行命令

## 测试结果

## 失败项

## 已知问题

## 未解决假设

## 修复建议

## 是否可以进入收尾阶段
```

## Outputs

- `docs/test-report.md`

## Completion Criteria

- `docs/test-report.md` exists.
- all run commands are recorded.
- results include pass/fail status.
- failures and known issues are explicit.
- readiness for closeout is stated.

## Stop Conditions

Stop and ask if:

- required validation commands cannot be determined.
- credentials, devices, or external services are required and unavailable.
- critical tests fail and the user asked not to fix them in this stage.
