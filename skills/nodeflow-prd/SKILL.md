---
name: nodeflow-prd
description: "Use when creating or updating a PRD before MVP, UI design, architecture, or implementation, especially from docs/BRIEF.md or a confirmed product idea."
---

# Nodeflow PRD

## Node Metadata

- Node ID: `nodeflow-prd`
- 作用: 从 BRIEF 或产品讨论生成 `docs/PRD.md`
- 输入: `docs/BRIEF.md`、用户产品信息、可选项目上下文
- 输出: `docs/PRD.md`
- 前置条件: BRIEF 已存在或用户已提供产品想法
- Gate: 产品目标、目标用户或核心功能不足以生成 PRD 时停止
- 默认 checkpoint: N；仅 human-guided 草案、覆盖文件或产品关键决策不清时需要
- 可跳过条件: 已有已确认 `docs/PRD.md`
- 下游节点: `nodeflow-architecture`, `nodeflow-change-spec`
- 适用风险: Medium / High

## Node Status

- Status: core
- Role: product-definition node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Stay in product definition. Do not design UI, architecture, database, MVP prototype, release plan, or implementation plan.
- Start from the user's own insight, pain point, and intended users.
- Put competitor analysis last. Do not let competitors define the product direction too early.
- Do not invent market facts, competitor details, user research, or business claims.
- Preserve uncertainty. Mark assumptions clearly.
- Let the user choose one-shot template mode or step-by-step module brainstorming mode. If the user provides an initial idea but does not choose, default to step-by-step mode.
- If the current project has product context in files such as `README.md`, `AGENTS.md`, `docs/`, or existing product notes, conservatively use that context to prefill the product definition before asking broad questions. Do not infer beyond the available evidence. Tell the user which files were used.
- Output each module draft as Markdown, separated from discussion text, so it can be copied into `PRD.md`.
- Output the prefilled module template as Markdown too, separated from guidance text.
- In human-guided mode, stop after producing the product definition draft and wait for user confirmation before writing any file.
- In agent-autonomous mode, generate `docs/PRD.md` from `docs/BRIEF.md` directly when the nodeflow requests it. Do not require an extra draft confirmation before writing.
- After agent-autonomous generation, do not decide whether to pause or continue. Follow `docs/NODEFLOW.md` checkpoint rules if present.
- Generate the final Markdown file at the current project's `docs/PRD.md` by default; if the user specifies another folder, use that folder instead.
- When generating `PRD.md`, use normal Markdown paragraphs and lists. Do not indent body content by four spaces, because that renders as code blocks in Markdown.
- Final `docs/PRD.md` must use a flat development-oriented structure, not `For Human / For Agent`.
- The final `docs/PRD.md` must include a dedicated `## 交互设计与页面流向 (Interaction Flow & App Surface)` section mapping the primary views/screens, transition states (e.g. Loading, Empty, Error, Success, Submitting), and key user flow steps.
- For high-risk, irreversible, external, or costly actions (such as deletion, data sync, or payments), establish explicit confirmation layers or approval gates in the interaction flow to prevent user errors.
- Extract development-relevant conclusions from background context into final PRD sections. Background context is useful only when it changes scope, core features, acceptance, exclusions, assumptions, or open questions.

## Checkpoint Focus

When this node reaches a checkpoint, the `Human Check` should focus on:

- product goal, target users, and core scenarios
- must-have features versus non-goals
- success standards and failure standards
- assumptions or open questions that would change implementation scope

## Auto Execution Rule

Generate `docs/PRD.md` directly when all conditions are true:

- The user explicitly asks to generate/create/write PRD, or `nodeflow` / `docs/NODEFLOW.md` routes here.
- `docs/BRIEF.md` exists, or the conversation contains enough product context to define product goal, target user, and core functionality.
- The action only creates or updates a local Markdown PRD.
- The target `PRD.md` does not already exist, or overwrite/append behavior is already specified.
- Missing details can be recorded as `需要确认` or `不确定点` without blocking the PRD.

Ask one focused question before writing when:

- product goal, target user, or core functionality is unclear.
- target `PRD.md` already exists and overwrite/append behavior is not specified.
- the requested PRD requires unsupported market facts, legal/business claims, or product decisions not present in the source material.
- the user explicitly asks for human-guided brainstorming rather than direct PRD generation.

## Confirmation Shorthand

- When waiting for confirmation, treat `1` as confirmation to continue with the current pending step.
- When asking the user to choose from numbered options, treat `1` as selecting option 1 instead.
- In user-facing prompts, explicitly say that replying `确认` or `1` continues to the next step.

## Template Prefill Rule

- If the user only invokes this skill and there is no usable project context, ask whether to use one-shot mode or step-by-step mode.
- If the user only invokes this skill but the current project has usable product context, conservatively prefill from project files and default to step-by-step mode unless the user explicitly chooses one-shot mode.
- If the user invokes this skill with an initial idea, conservatively prefill only fields clearly supported by the user's text.
- Leave uncertain fields empty. Do not write `待补充`.
- Do not infer target users, competitor facts, feature scope, or motivations that the user did not state.
- After prefill, start from `设计灵感` and guide the user module by module unless the user explicitly chooses one-shot mode.

## Module Skip Rule

- If the user explicitly asks to skip a module, keep that module heading in the draft and mark it as intentionally skipped or temporarily undefined.
- Add the skipped module to `不确定点` unless the user says it should be permanently excluded.
- Continue to the next module without treating the skipped content as confirmed evidence.

## Progress State Rule

- At the start of a resumed conversation, identify the active module, confirmed modules, unconfirmed modules, and next step from the conversation or existing draft.
- If the state is unclear, summarize the most likely state and ask the user to confirm before continuing.
- Do not re-ask already confirmed modules unless new evidence changes the product definition.

## Module Boundary Correction Rule

- If the user points out a missing layer, unclear responsibility, or mixed concern in the product flow, pause the current module and reason from first principles.
- Revise the module boundary before continuing. For example, split `data -> strategy` into `data -> processing/features -> strategy/signal` when data cleaning, feature extraction, and decision logic should be separate.
- Treat the corrected boundary as the active draft after the user confirms it.

## Draft Layering Rule

For human-guided product definition drafts, use `For Human / For Agent` as a review layer before writing the final file. Use `references/prd-templates.md` for the exact draft template.

Do not delete background modules in the draft. Compress them and avoid treating them as implementation requirements unless they explicitly affect scope, acceptance, or constraints.

## Final PRD Structure Rule

Final `docs/PRD.md` should be easy for later nodeflow stages to consume. Use `references/prd-templates.md` for the exact final file structure and section rules.

## Background Extraction Rule

Before producing the final draft or file, review background context and extract only development-relevant conclusions into the final PRD:

- From `设计灵感`: extract product direction only if it affects the product goal or scope.
- From `痛点定义`: extract must-solve problems, success standards, and failure standards.
- From `用户画像`: extract usage context, platform/device constraints, permission needs, and behavior constraints.
- From `竞品拆解`: extract `可借鉴点` into `核心需求` or `成功标准` only when it fits the user's brief; extract `不跟随点` into `非目标`.
- From `假设`: extract risky assumptions into `不确定点` or `需要确认`.

Keep the original background sections short. Do not duplicate long background text in development-facing PRD sections.

## Agent Autonomous Mode

Use this mode when:

- `docs/BRIEF.md` exists and the user asks to generate PRD from it.
- `docs/NODEFLOW.md` reaches product definition / PRD generation.
- An orchestrator asks the agent to read `docs/BRIEF.md` and create `docs/PRD.md`.

Steps:

1. Read `docs/BRIEF.md`.
2. If present, also read `AGENTS.md`, `CLAUDE.md`, and `docs/NODEFLOW.md` for project rules and checkpoint behavior.
3. Generate `docs/PRD.md` directly using the flat final PRD structure.
4. Include the nodeflow-required fields when applicable: 产品目标、目标用户、核心场景、必做功能、不做什么、成功标准.
5. Put uncertainty into `需要确认` or `不确定点`; if `docs/ASSUMPTIONS.md` already exists and the nodeflow requires it, add non-blocking assumptions there.
6. Do not perform competitor research by default. Run it only if `docs/BRIEF.md` or the nodeflow explicitly asks for competitor analysis.
7. After generation, follow `docs/NODEFLOW.md` checkpoint rules. If no nodeflow checkpoint is available, summarize the generated PRD and ask the user to confirm the next step.

Do not ask broad clarification questions in this mode unless `docs/BRIEF.md` is too vague to define product goal, target user, or core functionality.

## Phase 0: Choose Work Mode

If the user only invokes the skill without an initial idea, ask them to choose:

```markdown
请选择产品定义方式：

1. 一次性填写：我给完整模板，你一次性补充所有模块。
2. 逐模块头脑风暴：我们按 设计灵感 -> 痛点定义 -> 用户画像 -> 核心功能 -> 产品边界 / 暂不做什么 -> 竞品拆解 逐个讨论，每个模块满意后进入下一个。

回复选项编号即可；回复 `1` 表示选择一次性填写。
```

If the user already indicates a preference, follow it directly.

If the user invokes the skill with an initial idea and no mode preference, default to step-by-step mode. First show the conservatively prefilled template, then begin with `设计灵感`.

## Phase 1A: One-Shot Template Mode

If the user chooses one-shot mode or provides all modules at once, provide this template and ask the user to confirm or complete it:

```markdown
请确认或补充下面产品定义模板：

1. 设计灵感：
2. 痛点定义：
3. 用户画像：
4. 核心功能：
5. 产品边界 / 暂不做什么：
6. 竞品拆解：

确认后回复 `确认` 或 `1`，我会整理产品定义草案。
```

Field guidance:

- 设计灵感：想法来源、触发场景、最想保留的直觉。
- 痛点定义：用户遇到的问题、为什么值得解决、现有方式哪里不够好。
- 用户画像：目标用户、使用场景、限制、动机、习惯。
- 核心功能：必须解决什么、第一版需要哪些功能、哪些先不做。
- 产品边界 / 暂不做什么：非目标用户、暂不覆盖的场景、明确不做的能力。
- 竞品拆解：放最后，主要由 Agent 调研完成；用户可提供已知竞品或限制。

## Phase 1B: Step-By-Step Brainstorming Mode

If the user chooses step-by-step mode:

1. First show the conservatively prefilled module template if the user provided an initial idea.
2. Work on only one module at a time, starting with `设计灵感`.
3. Ask focused questions or offer 2-3 possible angles when useful.
4. Summarize the current module.
5. Ask the user whether this module is satisfactory.
6. Move to the next module only after the user confirms.
7. Keep competitor analysis last.

Module order:

1. 设计灵感
2. 痛点定义
3. 用户画像
4. 核心功能
5. 产品边界 / 暂不做什么
6. 竞品拆解

Use this output shape for each module:

```markdown
## 当前模块：{模块名}

## 我对当前信息的理解

## 可补充的问题 / 方向

## 本模块草案

```markdown
### {模块名}

- ...
```

请确认这个模块是否满意。回复 `确认` 或 `1` 后进入下一个模块。
```

Do not continue to the next module without confirmation.

For the initial step-by-step response with an initial idea, use:

```markdown
## 已根据初步想法预填

```markdown
1. 设计灵感：
2. 痛点定义：
3. 用户画像：
4. 核心功能：
5. 产品边界 / 暂不做什么：
6. 竞品拆解：
```

## 当前模块：设计灵感

## 我对当前信息的理解

## 可补充的问题 / 方向

## 本模块草案

```markdown
### 1. 设计灵感

- ...
```

请确认设计灵感是否满意。回复 `确认` 或 `1` 后进入痛点定义。
```

## Phase 2: Product Definition Draft

After the user completes one-shot mode or confirms all step-by-step modules:

1. Organize the idea into a concise product definition.
2. Separate facts, assumptions, and open questions.
3. Keep competitor analysis after inspiration, pain point, user, and core feature sections.
4. Do not propose MVP, business model, UI, architecture, or implementation unless the user asks for the next stage.
5. Ask the user to confirm the draft. Tell the user the default output path is `docs/PRD.md` unless they specify another folder.

Use this output shape:

Use the Human-Guided Draft template in `references/prd-templates.md`.

For `建议下一步`, recommend one of:

- continue refining product definition;
- switch to a later nodeflow for user interaction flow design;
- switch to a later nodeflow for MVP prototype validation;
- use another nodeflow skill only when appropriate.

Do not expand the next-stage nodeflow inside this skill.

## Phase 3: Generate PRD.md

In agent-autonomous mode, write the file directly when the Auto Execution Rule is satisfied.

In human-guided mode, only write a file after the user explicitly confirms the draft, such as:

- `确认，生成 PRD`
- `内容没问题`
- `确认，生成到 docs/`
- `内容没问题，输出到 <project-root>\docs`
- `保存为 PRD.md 到指定文件夹`

Rules:

- File name must be `PRD.md`.
- File format must be Markdown.
- Use normal Markdown paragraphs and lists; do not four-space-indent body content.
- Default output path is the current project's `docs/PRD.md`.
- If the user specifies another folder, write `PRD.md` to that folder instead.
- Before writing, check whether the target folder and target `PRD.md` already exist.
- If the target folder does not exist, create only that folder as needed.
- If target `PRD.md` already exists, do not overwrite silently. Ask whether to overwrite, append, or choose another folder.
- If the user chooses append, add a dated section or clearly separated revision section instead of duplicating the same headings silently.
- Do not create extra files.
- Do not add sections outside this skill's scope unless the user explicitly asks.

Use the final `docs/PRD.md` structure in `references/prd-templates.md`.

After writing the file, output:

```markdown
## 已生成文件

## 内容摘要

## 后续建议
```

## Competitor Analysis Rule

Competitor analysis is late-stage and mainly agent-led:

- Run competitor analysis only after `设计灵感`、`痛点定义`、`用户画像`、`核心功能`、`产品边界 / 暂不做什么` are confirmed.
- Research competitors directly when entering this module; no separate permission is required.
- If the user provides competitor names, start from them.
- If the user does not provide competitors, search for likely competitors based on the product category and target user.
- Include source links and the research date in competitor analysis. Base conclusions only on verifiable public information or clearly marked assumptions.
- If reliable public information cannot be found, say so explicitly and leave the unsupported part in `不确定点`.
- Analyze each competitor's target user, core features, strengths, weaknesses, and what to borrow or avoid.
- Do not make product direction depend on competitors alone.
- Use competitor analysis to clarify differentiation, avoid blind spots, and identify what not to copy.

Use this output shape:

```markdown
## 当前模块：竞品拆解

## 调研范围

## 调研来源

## 竞品列表

## 竞品优点

## 竞品缺点

## 可借鉴点

## 不跟随点

## 对当前产品定义的校准

## 本模块草案

```markdown
### 6. 竞品拆解

#### 竞品列表

#### 竞品优点

#### 竞品缺点

#### 可借鉴点

#### 不跟随点

#### 对当前产品定义的校准
```

请确认竞品拆解是否满意。回复 `确认` 或 `1` 后我会整理完整产品定义草案。
```

## Stop Conditions

Stop and ask before continuing if:

- The user asks for MVP, UI, architecture, implementation, release, or business validation in the same step.
- The product idea is too broad to define core users or core features.
- The requested output would require inventing user research or market evidence.
- The user asks to generate `PRD.md` but has not confirmed the draft.
- `PRD.md` already exists and the user has not approved overwrite or append.
