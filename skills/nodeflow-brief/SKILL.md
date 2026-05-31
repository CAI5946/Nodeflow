---
name: nodeflow-brief
description: "Use when turning a rough idea into docs/BRIEF.md, choosing an init profile, and bootstrapping folders before PRD, planning, or execution."
---

# Nodeflow Brief

## Node Metadata

- Node ID: `nodeflow-brief`
- 作用: 新项目 Intake 概念澄清，选择 Init Profile 并自动初始化目录结构与 Code Root
- 输入: 用户初步想法、项目路径、可选 init profile、可选 code root
- 输出: `docs/BRIEF.md`, `docs/DECISIONS.md`, `docs/CHANGELOG.md`, `docs/NODE_MAP.md`, 自动识别并记录的 Code Root
- 前置条件: 用户启动 0 -> 1 新项目开发或独立规划，且目标路径安全
- Gate: 目标路径不安全、目标文件存在且未指定覆盖/追加、项目目标极度模糊无法判定 `做什么`、或在 `new-generic-code` 下未确认技术栈前停止
- 默认 checkpoint: Y
- 可跳过条件: 已有可用 `docs/BRIEF.md` 且项目已完成初始化
- 下游节点: `nodeflow-prd`, `nodeflow-request-router`, `nodeflow-plan`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: 0-to-1 starter and intake node
- Replacement: none

## Core Rules

- Reply in Chinese.
- This skill is a nodeflow starter and concept clarifier. Do not generate full PRD, ARCH, TASKS, implementation code, or tests here.
- Preserve technical stack and engineering guards (N1):
  - Do not assume `src/` is the default code root. Flutter uses `lib/`; Android uses `app/`; Py/JS check existing codebase structure.
  - For new code projects with unclear stacks (`new-generic-code`), ask exactly one targeted question to confirm the technology stack and code root BEFORE creating code directories.
- Business documents go under `docs/`.
- Do not overwrite existing files silently. Always ask for overwrite or append approval.

## Init Profile Routing

Before drafting the BRIEF or bootstrapping, choose the smallest matching init profile:

```plain text
docs-maintenance
- nodeflow, skill, docs, research, or process maintenance
- create docs state only; no code root required

existing-codebase
- existing repo with framework files or code
- detect code root from existing files; do not create code directories

new-flutter-app
- user explicitly wants a new Flutter app
- expected code root: lib/
- prefer Flutter scaffold when project creation is in scope

new-generic-code
- new code project but stack/framework is unclear
- stop and ask for stack/code root before creating code directories

product-planning-only
- PRD, architecture, or task planning without implementation
- create docs state only; no code root required yet
```

---

## Flow

### Phase 1: Clarify & Choose Profile

If the user gives only a rough idea, ask targeted questions to choose the profile and clarify scope.

Ask the user to answer these 4 fields:
1. 做什么：目标是什么？什么场景下使用？
2. 必须有：不可妥协的核心功能？
3. 不要：明确不想要的内容（如：不要 GUI、不要云同步）？
4. 约束：运行平台、语言、依赖限制？

### Phase 2: Draft BRIEF & Identify Profile

After clarifications, output the `docs/BRIEF.md` draft and specify the detected **Init Profile** and **Code Root**.

Draft Structure:
```markdown
# BRIEF

## 做什么

## 必须有

## 不要

## 约束
```

Also output:
```markdown
## 初始化决策
- 判定 Profile: <selected-profile>
- 预估 Code Root: <detected-or-estimated-code-root>
```

### Phase 3: Bootstrap Folders & Save BRIEF

Once the user approves the BRIEF draft (replying `1` or `确认`):

1. **Write `docs/BRIEF.md`** automatically.
2. **Initialize Workspace State**:
   * Create `docs/` directory.
   * Generate `docs/DECISIONS.md` recording the chosen Profile and Code Root.
   * Generate `docs/CHANGELOG.md` with an initial record of the project bootstrap.
   * Generate `docs/NODE_MAP.md` representing the default 14-node structure.
3. **Identify Code Root (N1 Guard)**:
   * Flutter: `pubspec.yaml` + `lib/` -> code root is `lib/`.
   * Android: `settings.gradle` + `app/` -> code root is `app/`.
   * Next.js: `package.json` + `app/`/`pages/`/`src/`.
   * React/Vite: `package.json` + `src/`.
   * Python: package structure.
   * Generic: ask user or detect framework markers.

Output completion report:
```markdown
已生成：`docs/BRIEF.md` 及基础状态文件 (`docs/DECISIONS.md`, `docs/CHANGELOG.md`, `docs/NODE_MAP.md`)。
初始化 Profile: [Profile Name]
Code Root 记录成功: [Code Root Path]

验证：已回读文件，结构完整。
```

---

## Stop Conditions

Stop and ask before writing if:
- the requirement is too vague to write a usable BRIEF.
- target files already exist and overwrite behavior is not specified.
- the profile is `new-generic-code` and the technology stack/code root is unclear.
- the user tries to jump into PRD, plan, or execution before BRIEF confirmation.
