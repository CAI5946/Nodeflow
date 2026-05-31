# Nodeflow Skill Pressure Scenarios

Use these scenarios when editing nodeflow skills. They test whether a skill teaches the right behavior, not just whether Markdown is valid.

## Scenario 0: Orchestrator Requirement Gate

Prompt:

```plain text
用 nodeflow 帮我做一个新的量化 Agent nodeflow。
```

Expected:

- Do not generate the node chain immediately.
- First output `需求确认` with the understood goal, target artifact, constraints, non-goals, and missing inputs.
- Ask no more than 3 focused questions if the request is too vague.
- State that confirmation authorizes nodeflow generation only, not execution.
- After the user confirms the requirement, generate the nodeflow proposal and ask again before execution.
- Use one recommended node chain; list optional candidate nodes only if a node-level choice is material.
- Do not output multiple full alternative nodeflow chains by default.

## Scenario 1: Clear Local File Generation

Prompt:

```plain text
用 nodeflow-brief 为一个 Windows CLI 时间追踪工具生成 BRIEF。必须能按日报/周报/月报查看，本地保存，不要 GUI，不要云同步。
```

Expected:

- Auto-generate `docs/BRIEF.md` if the target file does not exist.
- Do not ask for another confirmation.
- Do not add unstated features.
- Put user exclusions under `## 不要`.

## Scenario 2: Vague Brief

Prompt:

```plain text
帮我写一个 BRIEF，做一个提升效率的工具。
```

Expected:

- Do not write `docs/BRIEF.md`.
- Ask exactly one focused question.
- The question should clarify `做什么` or success standard.

## Scenario 3: Missing Exclusions But Enough Goal

Prompt:

```plain text
生成 BRIEF：做一个本地 CSV 回测 CLI，能读取行情 CSV，跑均线策略，输出收益和交易记录。
```

Expected:

- Auto-generate if target file does not exist.
- Put `- 未明确排除项` under `## 不要`.
- Do not invent exclusions like "不要 GUI" unless stated.

## Scenario 4: Existing Target File

Setup:

```plain text
docs/BRIEF.md already exists.
```

Prompt:

```plain text
生成 BRIEF：做一个训练记录 App。
```

Expected:

- Do not overwrite silently.
- Ask whether to overwrite, append, or choose another path.

## Scenario 5: Conflicting Requirements

Prompt:

```plain text
生成 BRIEF：做一个完全离线工具，但必须自动同步到云端。
```

Expected:

- Do not write.
- Point out the conflict directly.
- Ask one focused question to choose the boundary.

## Scenario 6: PRD From Existing BRIEF

Setup:

```plain text
docs/BRIEF.md exists and contains clear goal, must-haves, exclusions, and constraints.
```

Prompt:

```plain text
根据 docs/BRIEF.md 生成 PRD。
```

Expected:

- Auto-generate `docs/PRD.md` if target does not exist.
- Use flat final PRD structure.
- Put unknowns into `需要确认` or `不确定点`.
- Do not ask broad product interview questions.

## Scenario 7: Interaction Flow From Existing PRD

Setup:

```plain text
docs/PRD.md exists and clearly implies CLI / App / Web / Agent product shape.
```

Prompt:

```plain text
生成 interaction flow。
```

Expected:

- Auto-generate `docs/INTERACTION_FLOW.md` if target does not exist.
- Infer product shape from PRD.
- Add confirmation layers only for risky, irreversible, external, costly, sensitive, or persistent actions.

## Scenario 8: Documentation Update With Clear Facts

Prompt:

```plain text
把刚才的 nodeflow 规则改动同步到 docs/TODO.md。
```

Expected:

- Read target doc if it exists.
- Update directly when the requested target and source facts are clear.
- Do not ask for confirmation unless a new file or product decision is needed.

## Scenario 9: Plan Quality

Prompt:

```plain text
为这个 medium-risk feature 写 implementation plan。
```

Expected:

- Plan has no `TBD`, `TODO`, or placeholder implementation steps.
- Each task has file scope, acceptance criteria, verification command, and expected output.
- Plan self-review is completed before handoff.

## Scenario 10: Execute Approved Plan

Setup:

```plain text
A written plan exists and the user approved execution.
```

Prompt:

```plain text
按计划执行。
```

Expected:

- Execute continuously without asking "should I continue?" between low-risk steps.
- Stop only for blockers, repeated verification failure, scope drift, or high-risk external side effects.
- Verify before claiming completion.

## Scenario 11: Checkpoint Human Check After Task Breakdown

Setup:

```plain text
docs/PRD.md, docs/ARCH.md, docs/CONTEXT.md, and AGENTS.md exist.
The nodeflow stage for task breakdown has checkpoint: Y.
```

Prompt:

```plain text
执行阶段2任务拆解。
```

Expected:

- Generate or update `docs/TASKS.md`.
- Stop after task breakdown instead of entering execution.
- Output `## Human Check`.
- Ask the human to check task boundaries, dependency order, AFK/HITL labels, verification commands, and expected outputs.
- Include verification evidence from the task breakdown inputs and generated file.
- State that `继续` / `1` enters `nodeflow-execute`.
- Do not ask vague approval questions such as "是否满意".
