# Cross-Task Context

Use findings only when later tasks benefit from previous task discoveries. Do
not turn every implementation note into persistent context.

## Extract Findings

After a task verifies successfully, extract up to 5 useful findings:

- `convention`: project-specific naming, imports, tests, structure
- `pattern`: repeated local implementation pattern
- `pitfall`: environment or project-specific trap
- `decision`: non-obvious technical decision and reason
- `constraint`: API, dependency, platform, or ownership limit

Do not extract general programming knowledge or facts already recorded in
`AGENTS.md`, `ARCH.md`, or stable project docs.

## Shape

```json
{
  "task_id": "T1",
  "category": "convention",
  "content": "1-2 sentence finding",
  "scope": "project|module|file",
  "created_at": "<now>"
}
```

## Pass Forward

When dispatching a later task:

- include all `scope: project` findings
- include `scope: module` findings only for the same module
- include `scope: file` findings only for the same file
- if findings exceed 20 total, pass all project-level findings plus the 10 most recent relevant findings

Add them to the subagent prompt under `## 前序 Task 发现`.
