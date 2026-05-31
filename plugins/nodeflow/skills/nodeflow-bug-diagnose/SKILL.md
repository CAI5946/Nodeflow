---
name: nodeflow-bug-diagnose
description: "Use before fixing unclear bugs, errors, crashes, regressions, test failures, or runtime symptoms to find evidence and root cause."
---

# Nodeflow Bug Diagnose

## Node Metadata

- Node ID: `nodeflow-bug-diagnose`
- 作用: 在修 bug 前复现问题并定位 root cause
- 输入: bug 描述、错误输出、日志、`AGENTS.md`, 相关代码
- 输出: 复现结果、根因证据、修复方向、验证命令
- 前置条件: 用户请求 bugfix 或询问失败原因
- Gate: 未复现或无 root cause 证据时不直接修复
- 默认 checkpoint: Y
- 可跳过条件: Low 风险且根因已由可靠输出直接证明
- 下游节点: `nodeflow-plan`, `nodeflow-execute`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: diagnosis node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Diagnose before fixing.
- Do not guess a fix without evidence.
- Prefer real command output, logs, tests, and minimal reproduction.
- Separate observation, hypothesis, experiment, and conclusion.
- Treat "cannot reproduce" as a valid diagnostic result, not permission to invent a fix.
- Prefer a failing test or minimal repro before implementation when practical.
- If the issue cannot be reproduced, say so and record what evidence is missing.
- Remove temporary instrumentation after use.

## Inputs

- user bug report or error
- logs, screenshots, stack traces, or failing command output
- `AGENTS.md`
- relevant source files
- test or run commands

## Process

```plain text
1. Observe: collect exact symptom, command, output, logs, environment, and recent change context.
2. Reproduce: run the failing path or create a minimal repro.
3. Stabilize feedback: choose the fastest repeatable check.
4. Hypothesize: list 1-3 plausible causes tied to evidence.
5. Test hypotheses: inspect code, add temporary instrumentation, or run targeted commands.
6. Identify root cause: explain why this cause produces the symptom and why alternatives are less likely.
7. Write failing test or repro case when practical.
8. Propose fix path and verification command.
9. Clean temporary diagnostics after use.
10. Record post-mortem when useful.
```

## Output Shape

```markdown
## 诊断结论

- 复现命令：
- 复现结果：
- 根因：
- 证据：
- 修复方向：
- 验证命令：
- 风险：
```

## Gate

```plain text
Do not directly fix a bug before reproduction or root-cause evidence unless the user explicitly asks for a best-effort workaround.
```

## Red Flags

Stop and gather evidence if you catch yourself thinking:

- "This error looks familiar, so the fix is obvious."
- "I'll change one thing and see what happens" without a hypothesis.
- "I cannot reproduce it, but I can still patch the likely area."
- "The stack trace mentions this file, so the root cause must be there."
- "The test is flaky, so I can ignore the failure."

## Stop Conditions

Stop and ask if:

- reproduction requires unavailable credentials, device, data, or service.
- logs or outputs are insufficient and no local reproduction path exists.
- the fix would affect data, release, account, payment, or architecture.
