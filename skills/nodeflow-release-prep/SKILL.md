---
name: nodeflow-release-prep
description: "Use for app release preparation, store readiness, signing, screenshots, privacy/compliance, build checks, or release checklists."
---

# Nodeflow Release Prep

## Node Metadata

- Node ID: `nodeflow-release-prep`
- 作用: 发布前检查、构建、素材、隐私/合规和上线准备
- 输入: 发布目标、平台、项目配置、`AGENTS.md`
- 输出: 发布检查结果、必要变更、验证报告
- 前置条件: 用户请求发布准备或上线检查
- Gate: 涉及版本、签名、商店文案、隐私或资产变更时需确认
- 默认 checkpoint: Y
- 可跳过条件: 当前任务不是发布或构建相关
- 下游节点: `nodeflow-execute`, `nodeflow-validate`, `nodeflow-handoff`
- 适用风险: High

## Node Status

- Status: specialized
- Role: release node
- Replacement: none

## Core Rules

- Reply in Chinese.
- Treat release work as high-risk.
- Check before changing.
- Do not generate legal claims, privacy statements, pricing, or store promises without user-provided facts.
- Do not expose or commit secrets, signing keys, API keys, or keystore files.
- Confirm before changing version, signing config, store text, or release assets.
- For Flutter projects, run Flutter commands in the app directory when needed.
- When reporting a release-blocking issue, include a reproduction flow when it can be reproduced by command, build step, store-console step, or device operation.
- After approved release changes, verification must include concrete manual steps; for app releases, include real-device smoke-test steps.

## Confirmation Shorthand

- When waiting for confirmation, treat `1` as confirmation to continue with the current pending step.
- When asking the user to choose from numbered options, treat `1` as selecting option 1 instead.

## Template Prefill Rule

- If the user only invokes this skill, provide an empty template.
- If the user invokes this skill with an initial idea, conservatively prefill only fields clearly supported by the user's text.
- Leave uncertain fields empty. Do not write `待补充`.
- Do not infer version numbers, legal/compliance content, signing configuration, platform, pricing, or release assets.
- Ask the user to confirm or complete the template before entering Phase 1.

## Phase 0: Initial Invocation

If details are missing, provide this template and ask the user to confirm or complete it:

```markdown
请按下面模板补充发布信息：

1. 发布目标：
2. 平台：
3. 应用名称：
4. 当前版本：
5. 发布类型：
6. 已准备材料：
7. 需要 Agent 做什么：
8. 不允许改动：
9. 验收标准：
```

## Phase 1: Readiness Assessment

Read relevant docs/configs only as needed. Output:

```markdown
## 问题 / 目标理解

## 修改必要性

## 最佳方案

## 需要修改的文件

## 不会改动的内容

## 风险与影响

## 验证方式

等待你确认后再执行。
```

Do not modify files in this phase.

## Phase 2: Execute Approved Release Work

Only edit or run build steps after explicit approval.

Possible checks:

- Privacy policy, user agreement, foreground service declaration if applicable.
- App icon, feature graphic, screenshots, store title, short description, full description.
- Version name/code.
- Signing config and `.gitignore` protection.
- Debug flags, hardcoded secrets, logging level.
- Crash/analytics readiness if used.
- Flutter build, such as `flutter build appbundle`.
- Closed testing checklist.

After execution, output:

```markdown
## 执行内容

## 修改文件

## 构建 / 检查结果

## 实机测试步骤

## 仍缺失内容

## 上架前人工确认

## 最终结论
```

## Stop Conditions

Stop and ask before continuing if:

- A secret, keystore, or signing config may be exposed.
- Store/legal content lacks factual input.
- Build requires credentials not available locally.
- Release commands fail and the next step could change signing, versioning, or store state.

## 基础发布清单

- 合规：隐私政策、用户协议、必要声明。
- 物料：图标、置顶图、截图。
- 文案：应用名、简短描述、完整描述。
- 打包：版本号、签名、release build。
- 检查：debug 关闭、敏感 key 未硬编码、日志级别合理。
- 测试：封闭测试或人工回归路径明确。
