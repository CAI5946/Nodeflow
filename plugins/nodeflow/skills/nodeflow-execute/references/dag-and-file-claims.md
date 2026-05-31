# DAG and File Claims

Use this only for planned task sets. Do not apply it to simple Low-risk work.

## DAG Scheduling

1. Read `dag.groups` from `docs/.nodeflow/<nodeflow_id>/state.json`.
2. Start from the first group with pending tasks.
3. Execute `parallel: false` tasks sequentially.
4. Dispatch same-group `parallel: true` tasks only when the selected execution mode allows it.
5. Skip `blocked` tasks until upstream blockers are resolved.

If tasks exist but no DAG has been computed, compute a simple topological order
from `dependencies`. Stop if there is a cycle.

## File Claims

Before parallel dispatch:

1. Register each task's declared `files` in `file_claims`.
2. Compare file sets for overlap.
3. If two tasks claim the same file, downgrade to serial execution or use worktree isolation.
4. Remove a task's claims after the task completes.

## Scope Review

During task-level review:

- Compare actual changed files with declared files.
- Reasonable expansions include new tests, type definitions, or directly required config.
- Unexplained edits outside task scope must be flagged and either reverted or approved.

## Failure Propagation

When a task fails:

1. Set the task status to `failed`.
2. Mark downstream tasks as `blocked`.
3. Add blockers with `reason: "upstream_failed:<task_id>"`.
4. Do not interrupt tasks already in progress; review them when they finish.

When the failed task is fixed and verified, unblock downstream tasks whose
dependencies are now satisfied.
