---
name: nodeflow-verify
description: "Use before claiming work is complete, fixed, working, or tested; requires command output, exit code, and coverage judgment."
---

# Verification Before Completion

## Node Metadata

- Node ID: `nodeflow-verify`
- 作用: 在声明完成前生成新鲜验证证据
- 输入: 验证命令、原始请求或 bug、当前代码状态
- 输出: 命令、输出摘要、exit code、失败数量、是否通过、是否覆盖原问题
- 前置条件: 任务准备声明完成
- Gate: 没有新鲜验证证据时不允许声明完成
- 默认 checkpoint: N
- 可跳过条件: 用户明确禁止运行命令，但必须说明未完整验证
- 下游节点: `nodeflow-handoff`, `nodeflow-validate`
- 适用风险: Low / Medium / High

## Node Status

- Status: core
- Role: verification node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Do not say "完成", "修好了", "测试通过", or equivalent without fresh verification evidence.
- Run the relevant command when available.
- Read the full output.
- Confirm exit code.
- Record failure count.
- State whether the check covers the original request.
- Do not create a verification report unless `document_budget=durable-evidence` or the user/project requires it.
- Treat old output, remembered output, or another session's output as stale unless re-run is impossible and clearly stated.
- If verification only partially covers the request, say exactly what remains unverified.
- If the task declared `capability_skills`, include the relevant observable check in the coverage judgment.

## Inputs

- verification command from plan, `AGENTS.md`, or project config
- original request or bug
- current code state
- prior test output if any
- `docs/.nodeflow/<nodeflow_id>/state.json`（如存在，用于写入验证结果）

## Process

1. Identify the most relevant verification command.
2. Identify any capability-specific evidence needed. For `frontend-design`, prefer screenshot or browser inspection; check desktop and mobile viewports when layout can change.
3. Run it.
4. Read the full output.
5. Record exit code.
6. Record failure count.
7. Decide whether it covers the original request and any declared capability skills.
8. Report evidence in the final answer or write it to `docs/CHANGELOG.md` when nodeflow requires it.
9. If `docs/.nodeflow/<nodeflow_id>/state.json` exists, update the current task's `verification` field with command、exit_code、failures、passed，更新 `last_heartbeat` 和 `updated_at`。

## Evidence Contract

Fresh verification evidence must include:

- command or manual check performed
- output summary based on current output
- exit code or observable result
- failure count
- coverage judgment against the original request
- capability-specific coverage when `capability_skills` is declared

If any item is missing, the handoff must say the work is not fully verified.

## Verification Report Policy

Do not create a report file by default. Write `docs/reports/YYYY-MM-DD-request-name-report.md` only when `document_budget=durable-evidence` or durable evidence is needed:

- release, build, signing, store, migration, security, privacy, or production risk
- multiple commands or manual flows must be preserved
- failures need follow-up tracking
- another agent or human will continue from the evidence
- the user explicitly asks to keep a verification report

For Low-risk and most Medium-risk work, keep verification evidence in the final reply and update `docs/CHANGELOG.md` only when the project nodeflow requires it.

## Output Shape

```markdown
## 验证结果

- 命令：
- 输出摘要：
- exit code：
- 失败数量：
- 是否通过：
- 是否覆盖原始问题：
```

## If Verification Cannot Run

Say exactly why:

- missing dependency
- unavailable service
- missing credentials
- command unknown
- environment limitation

Then provide the best alternative check, but do not claim full verification.

## Red Flags

Stop and correct the verification story if you catch yourself thinking:

- "It should pass because the change is small."
- "The same command passed earlier."
- "No code changed, so no verification is needed."
- "The output is long; I can skim only the success-looking part."
- "A screenshot or file read is enough even though a project command exists."

## Auto-Retry Integration

当 nodeflow-execute 触发自动修复循环时：

1. Verify 节点接收修复后的代码
2. 重新运行验证命令
3. 返回验证结果给 execute 节点

### 角色定义

- Verify 节点**只负责验证并报告结果**，不负责修复
- 修复由 nodeflow-execute 节点负责
- Verify 节点需要提供详细的失败信息，以便 execute 节点定位问题

### 重试时的验证重点

当接收到重试请求时：

- 重新运行完整的验证命令
- 对比本次和上次的验证结果
- 如果仍失败，提供详细的错误输出和可能的原因

### 输出格式

重试时的验证结果应包含：

```markdown
## 验证结果

- 命令：
- 输出摘要：
- exit code：
- 失败数量：
- 是否通过：
- 是否覆盖原始问题：
- 重试次数：N/3
- 与上次对比：新增失败 / 修复失败 / 无变化
```

## Stop Conditions

Stop and ask if:

- verification would be destructive.
- required credentials or devices are unavailable.
- the user explicitly asked not to run commands.
