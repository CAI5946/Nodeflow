# Nodeflow State Management

Use state only when the nodeflow needs recovery, task tracking, checkpoints, or
cross-task context. Do not create state just to make Low-risk work look formal.

## Paths

- State directory: `docs/.nodeflow/<nodeflow_id>/`
- State file: `docs/.nodeflow/<nodeflow_id>/state.json`
- Active pointer: `docs/.nodeflow/active.json`

State files are project-local runtime state and must not be committed unless a
project explicitly decides otherwise.

## Initialize

After nodeflow approval:

1. Generate `nodeflow_id` as `YYYY-MM-DD-<slug>`.
2. Create `docs/.nodeflow/<nodeflow_id>/`.
3. Write `state.json`.
4. Update `active.json`.
5. If another active nodeflow is unfinished, ask whether to continue, replace the active pointer, or cancel.

Minimum shape:

```json
{
  "version": 3,
  "nodeflow_id": "YYYY-MM-DD-slug",
  "created_at": "<now>",
  "updated_at": "<now>",
  "last_heartbeat": "<now>",
  "mode": "<execution_mode>",
  "risk": "<risk>",
  "document_budget": "<document_budget>",
  "worktree_policy": "<worktree_policy>",
  "requirement_anchor": {
    "goal": "",
    "must_have": [],
    "must_not": [],
    "acceptance_criteria": [],
    "output": "",
    "confirmed_at": "<now>"
  },
  "chain": [],
  "current_node": "",
  "tasks": [],
  "dag": { "groups": [], "computed_at": "<now>" },
  "findings": [],
  "file_claims": {},
  "review_matrix": {},
  "checkpoints": [],
  "subagents": [],
  "blockers": [],
  "learn_mode": false
}
```

## Resume

At intake, check `active.json`:

1. If missing, start normally.
2. If present, load the pointed `state.json`.
3. If `last_heartbeat` is older than 30 minutes, treat the nodeflow as interrupted.
4. Mark any `in_progress` task as `needs_review`; do not auto-retry.
5. Summarize completed tasks, pending tasks, blockers, and current node.
6. Ask whether to continue, restart, or modify the nodeflow.

## DAG

For planned tasks:

- Build `dag.groups` from task dependencies.
- Put no-dependency tasks in group 0.
- A task enters group N when all dependencies are in earlier groups.
- Mark same-group tasks parallel only when declared file paths do not overlap.
- Stop if a cycle is detected.

## Failure Propagation

When a task fails:

1. Mark it `failed`.
2. Mark downstream tasks that depend on it as `blocked`.
3. Add `blockers` entries with `reason: "upstream_failed:<task_id>"`.
4. If the failed task is later fixed, resolve blockers whose dependencies are now satisfied.

## Iteration Loops

For each task, state may record review and retry data:

```json
{
  "task_id": "T1",
  "status": "in_progress|needs_review|completed|failed|blocked",
  "review_matrix": ["spec-review-agent", "code-quality-review-agent"],
  "review_findings": [],
  "retry_count": 0,
  "retry_history": []
}
```

Loop rules:

- `nodeflow-execute` may route a task back to execution after review or verification failure.
- Increment `retry_count` only when a fix attempt is made after a failed review or verification.
- Stop after 3 attempts and set `status: "needs_review"` or `failed`.
- Downstream DAG tasks remain `blocked` until the current task is reviewed and verified.
- `last_heartbeat` must update on every loop iteration.

## Checkpoints

After checkpoint approval, append:

```json
{
  "node": "<node_id>",
  "status": "approved",
  "approved_at": "<now>"
}
```

On resume, skip approved checkpoints.

## CURRENT.md

Generate or update `docs/CURRENT.md` only when the document budget or project
profile requires it. Keep it short: current node, active task, completed tasks,
next task, blockers.
