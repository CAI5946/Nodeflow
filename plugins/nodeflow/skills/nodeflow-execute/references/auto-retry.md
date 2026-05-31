# Auto Retry Policy

Auto retry applies after review or verification failure when the issue is local,
in scope, and fixable. P1/P2 review findings do not automatically stop the
task; fix them when the expected behavior is clear and the fix stays inside the
approved scope. Stop for `AskHuman` when the finding changes product behavior,
architecture, external state, or approved scope.

## Flow

1. Run the review matrix from `review-loop.md`.
2. If P1/P2 findings exist, classify whether each finding is fixable in scope.
3. Fix in-scope findings and re-run the relevant review profiles.
4. Run `nodeflow-verify`.
5. If verification passes and reviewers pass, mark the task complete.
6. If review or verification still fails, classify whether the failure is auto-fixable.
7. Retry up to 3 times.
8. Before the third attempt (`retry_count = 2`), ask through `AskHuman`.

## Auto-Fixable

- syntax or parse errors
- import or module path errors
- type errors
- assertion failures
- lint or formatting errors
- simple local logic errors
- review findings with clear expected behavior and declared file scope
- missing tests or observable checks that can be added without changing scope

## Not Auto-Fixable

- missing dependencies or broken environment
- credentials, permissions, devices, or external services
- unclear product behavior
- architecture or API design problems
- performance/resource failures without a local fix path
- findings that require changing approved scope, non-goals, or product behavior

## Retry History

Record each attempt:

```json
{
  "attempt": 1,
  "trigger": "verification_failed",
  "issue": "short failure summary",
  "fix_applied": "short fix summary",
  "result": "fixed|still_failing",
  "auto_fixable": true,
  "timestamp": "<now>"
}
```

## AskHuman Prompt

```markdown
验证已失败 2 次：
- 第 1 次：
- 第 2 次：

是否继续尝试第 3 次修复？回复 `继续` / `1` 或 `停止` / `2`。
```
