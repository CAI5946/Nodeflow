---
name: nodeflow-execute
description: "Use to execute approved nodeflow tasks, respect HITL pauses, preserve user changes, verify, and update docs only when required."

---

# Nodeflow Execute

## Node Metadata

- Node ID: `nodeflow-execute`
- 作用: 按已批准范围或任务计划执行修改
- 输入: `AGENTS.md`, 已批准请求 / approved plan, 相关文件
- 输出: 代码/文档变更、验证证据、可选进度/变更记录
- 前置条件: 任务范围、依赖和验证命令已明确
- Gate: HITL 任务、依赖未满足、文件冲突或缺少验证命令时停止
- 默认 checkpoint: N
- 可跳过条件: 请求只需要调研或决策，不需要修改文件
- 下游节点: `nodeflow-review`, `nodeflow-verify`, `nodeflow-validate`
- 适用风险: Low / Medium / High

## Node Status

- Status: core
- Role: execution node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Execute only tasks defined in an approved plan or a confirmed Low-risk request.
- Follow `AGENTS.md` for commands, paths, and repository rules.
- Preserve user changes; do not overwrite unrelated work.
- Do not skip dependencies.
- Do not execute HITL tasks without human confirmation.
- After a plan is approved, execute eligible AFK tasks continuously. Do not ask "should I continue?" between low-risk steps.
- Follow the selected `execution_mode`; do not start subagents or worktrees unless the mode/policy allows it.
- For complex implementation or parallel tasks, prefer isolated worktrees unless the plan records an exception.
- If the approved request or plan declares `capability_skills`, load those skills before editing files. They provide specialist execution rules but do not change the Nodeflow chain.
- Do not create or update progress docs by default for Low-risk work; respect `document_budget`.
- After every task that changes files or behavior, run the review matrix from `references/review-loop.md`.
- Treat review/verification as an execute-check loop: fix in-scope P1/P2 findings and failed checks, then re-run the relevant reviewers, up to the retry limit.
- Run or invoke verification after changes.
- Do not claim completion until `nodeflow-verify` has fresh evidence.

## Inputs

Required:

- `AGENTS.md`
- approved request or approved plan
- verification command or a way to derive it from project config

Recommended when present:

- `docs/.nodeflow/<nodeflow_id>/state.json`
- `docs/CURRENT.md`
- `docs/CONTEXT.md`
- `docs/ARCH.md`
- `docs/DECISIONS.md`
- `docs/CHANGELOG.md`

## Execution Order

1. Read `AGENTS.md` and the approved request or plan.
2. Read state if it exists; resume from the next pending task whose dependencies are satisfied.
3. Confirm `document_budget`, `execution_mode`, and `worktree_policy`.
4. Read `capability_skills` from the plan or request. If present, load the listed specialist skills before modifying files.
5. Determine whether the work is a single direct task or a planned task set.
6. For planned tasks, follow DAG and file-claim rules from `references/dag-and-file-claims.md`.
7. Complete shared foundation work before any parallel group.
8. Stop before HITL tasks and ask through `AskHuman`.
9. Execute one task at a time unless the approved mode allows parallel dispatch.
10. After each task:
   - run the required review matrix from `references/review-loop.md`
   - run `nodeflow-verify` or the approved verification command
   - if review or verification fails, classify fixability and use `references/auto-retry.md`
   - loop back to task execution for in-scope fixable failures
   - update state only when state exists or the document budget requires it
   - extract cross-task findings only when another task will use them
11. Continue until all eligible tasks in the current group are complete or a stop condition is reached.

## Execution Modes

- `simple-sequential`: execute the confirmed Low-risk scope directly, then verify and hand off.
- `planned-sequential`: execute plan tasks in dependency order in the current workspace.
- `subagent-driven`: dispatch independent tasks only when file ownership is non-overlapping and merge/review is defined.
- `worktree-subagent`: use isolated worktrees for complex or parallel tasks; merge results only after review and verification.

## Per-Task Review Matrix

- Low risk docs/copy/config-only changes: run at least `verification-review-agent` when there is an observable check; add `code-quality-review-agent` when files changed.
- Medium / High planned tasks: run `spec-review-agent`, `code-quality-review-agent`, and `verification-review-agent`.
- Capability work: add `capability-review-agent`.
- Release, migration, security, privacy, production, account, payment, or external-state work: add `risk-review-agent`.
- Use real subagents only when available and allowed by `execution_mode`; otherwise run the same profiles sequentially in the controller session.

## Verification Discipline

Agent must not say "完成", "修复成功", or "测试通过" unless this happened in the current turn:

1. Run the verification command defined in the approved plan or derived from the project.
2. Read the output.
3. Confirm exit code.
4. Record failure count.
5. Judge whether the verification covers the request.

If verification cannot run, state why and provide the best alternative check. Do not claim full verification.

## Progress Docs Policy

- Low risk: do not update task docs by default.
- Medium risk: update plan/status docs only if they already exist or are needed for continuity.
- High risk or durable-evidence work: update the approved spec/plan/status docs according to the plan.
- Append `docs/CHANGELOG.md` only when the project nodeflow, existing plan, or `document_budget` requires durable change tracking.

## References

Read these only when needed:

- `references/dag-and-file-claims.md`: DAG group execution, file claims, conflict handling, failure propagation.
- `references/review-loop.md`: per-task review profiles, execute-check iteration, fixability rules.
- `references/auto-retry.md`: task-level review, verification failure retry policy, retry history.
- `references/cross-task-context.md`: findings extraction and context transfer between tasks.
- `../nodeflow/references/capability-skills.md`: ordinary specialist skills used inside plan, execute, review, or verify.
- `../nodeflow/references/state-management.md`: state initialization and recovery.

## Red Flags

Stop and rethink if you catch yourself doing any of these:

- "I'll also clean up nearby code while I am here."
- "The plan says execute, so I can ignore user changes in the worktree."
- "Parallel tasks probably won't touch the same files."
- "Verification failed, but the change looks obviously correct."
- "This doc update is Low risk, so no verification is needed."
- "I can update progress docs for every small step."

## Outputs

- changes within task scope
- verification evidence
- updated state file when present or required
- optional `docs/CURRENT.md` / `docs/CHANGELOG.md` updates when justified

## Completion Criteria

- all eligible tasks in the current group are done
- required review matrix is complete for each changed task
- `nodeflow-verify` result is recorded
- retry count/history are recorded if retry happened
- required progress or changelog docs are updated when applicable

## Stop Conditions

Stop and ask if:

- a task is marked HITL.
- dependencies are unsatisfied.
- two parallel tasks need the same core file.
- verification command is missing for a task that claims to be AFK.
- verification fails and the failure is not auto-fixable.
- verification fails 3 times.
- the user chooses to stop at retry_count = 2.
- implementation would drift beyond approved task scope.
- execution would affect external systems, accounts, releases, payments, secrets, privacy, production data, or irreversible state.
- state file write fails when state is required.
