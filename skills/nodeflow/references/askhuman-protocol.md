# AskHuman Protocol

`AskHuman` is a decision protocol, not a callable tool name.

Use it only when a human decision affects scope, routing, checkpoint approval,
learn mode, retry decisions, or execution safety.

## Adapter Order

1. Claude Code with `AskUserQuestion` available: use `AskUserQuestion`.
2. Codex Plan mode with `request_user_input` available: use `request_user_input`.
3. Other agents, Codex Default mode, or missing tool support: use plain text options.

## Plain Text Fallback

Ask one question. Provide 2-3 choices when possible, put the recommended option
first, and tell the user to reply with `1/2/3`, `确认`, `继续`, or `跳过`.

## Rules

- Do not treat `AskHuman` as an actual tool.
- Do not make `AskUserQuestion` or `request_user_input` a hard dependency.
- Do not claim Nodeflow can switch Codex into Plan mode.
- Persist checkpoint approvals and material recovery decisions to state.
- Do not persist every ordinary choice.

## Checkpoint Output

When a checkpoint is required, output a focused `Human Check`:

```markdown
## Human Check

- 你需要确认：
- 建议检查：
- 当前风险 / 假设：
- 验证证据：
- 回复 `继续` / `1` 后将进入：
```

Keep the checklist specific to the node output. Do not ask the human to re-check
evidence the agent can verify from files or command output.
