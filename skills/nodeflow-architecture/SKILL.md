---
name: nodeflow-architecture
description: "Use after docs/PRD.md exists to decide architecture and create ARCH, CONTEXT, and DECISIONS docs before task breakdown."
---

# Nodeflow Architecture

## Node Metadata

- Node ID: `nodeflow-architecture`
- 作用: 生成架构、领域词典、决策记录，并触发项目级 agent 规则
- 输入: `docs/PRD.md`、可选 `docs/BRIEF.md`, `docs/NODEFLOW.md`, `docs/ASSUMPTIONS.md`
- 输出: `docs/ARCH.md`, `docs/CONTEXT.md`, `docs/DECISIONS.md`, `AGENTS.md`, `CLAUDE.md`
- 前置条件: `docs/PRD.md` 已存在且核心需求明确
- Gate: 技术栈、模块边界、共享接口或数据模型不清时停止
- 默认 checkpoint: Y
- 可跳过条件: 已有已确认 `docs/ARCH.md` 和 `AGENTS.md`
- 下游节点: `nodeflow-task-breakdown`
- 适用风险: High

## Node Status

- Status: core
- Role: architecture node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Stay in architecture and project-rule preparation.
- Do not implement code or generate `TASKS.md`.
- Read `docs/PRD.md` before deciding architecture.
- Generate `AGENTS.md` only after architecture is defined enough to name tech stack, structure, commands, and validation expectations.
- Keep `docs/CONTEXT.md` as a domain glossary, not a requirements or architecture document.
- Put decisions in `docs/DECISIONS.md`; do not hide major tradeoffs inside prose.
- After architecture is defined, automatically load `write-agents-md` as a capability skill to generate `AGENTS.md`. Do not wait for external routing.

## Checkpoint Focus

When this node reaches a checkpoint, the `Human Check` should focus on:

- whether the chosen technical direction is the simplest viable option
- module boundaries, shared interfaces, and shared data models
- storage, external dependency, account, secret, and privacy boundaries
- decisions that would be expensive to reverse after task breakdown

## Inputs

Required:

- `docs/PRD.md`

Recommended if present:

- `docs/BRIEF.md`
- `docs/NODEFLOW.md`
- `docs/ASSUMPTIONS.md`
- existing repository files and config

## Actions

1. Read required inputs.
2. Identify product constraints that affect architecture:
   - platform
   - data boundary
   - interaction mode
   - storage needs
   - external dependencies
   - validation expectations
3. Review from four angles:
   - product fit
   - technical feasibility
   - data structure
   - risk
4. Generate `docs/ARCH.md`.
5. Generate `docs/CONTEXT.md`.
6. Update `docs/DECISIONS.md`.
7. Load `write-agents-md` as a capability skill and use it to create/update `AGENTS.md`.
8. Create `CLAUDE.md` with only:

```markdown
@AGENTS.md
```

## ARCH.md Structure

```markdown
# ARCH

## 技术栈

## 模块边界

## 数据格式

## 文件结构

## 核心依赖

## 风险点

## 共享接口

## 共享数据模型

## 验证方式
```

## CONTEXT.md Rules

`docs/CONTEXT.md` must contain only project-specific terms and concepts:

```markdown
# CONTEXT

## 术语

- Term: one-sentence definition.
```

Rules:

- one sentence per term
- no implementation details
- no requirements
- no architecture decisions
- code naming should stay consistent with these terms

## DECISIONS.md Rule

Append or update decisions with:

```markdown
## YYYY-MM-DD - Decision Title

- 决策：
- 理由：
- 影响：
- 替代方案：
```

## Outputs

- `docs/ARCH.md`
- `docs/CONTEXT.md`
- `docs/DECISIONS.md`
- `AGENTS.md`
- `CLAUDE.md`

## Completion Criteria

- `docs/ARCH.md` exists and covers tech stack, modules, data, dependencies, risks, interfaces, and shared models.
- `docs/CONTEXT.md` exists and contains only glossary content.
- `docs/DECISIONS.md` records key architecture choices.
- `AGENTS.md` exists and is based on project facts, not guesses.
- `CLAUDE.md` exists and points to `AGENTS.md`.

## Stop Conditions

Stop and ask if:

- `docs/PRD.md` is missing.
- the PRD does not define product goal or core requirements clearly enough to choose architecture.
- tech stack or platform choice is a high-impact decision and cannot be inferred.
