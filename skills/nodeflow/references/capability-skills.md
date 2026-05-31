# Capability Skills

Capability skills are ordinary specialist skills used inside Nodeflow tasks.
They are not Nodeflow nodes and must not be added to default chains unless they
become a reusable workflow stage with clear input, output, gate, and downstream
nodes.

## Rules

- Keep Nodeflow responsible for routing, risk, document budget, execution mode, and verification.
- Use capability skills only when the task domain needs specialist rules.
- Declare capability skills in the plan as `capability_skills`.
- Load declared capability skills before editing files.
- Do not change risk level, document budget, or execution mode just because a capability skill exists.
- Do not create a `nodeflow-*` node for a normal tool-like skill.

## Common Mapping

| Scenario | Capability skill | Use during | Trigger |
| --- | --- | --- | --- |
| Reference / competitor analysis | `reference-analysis` | intake / brainstorming / plan | product, competitor, UX, pricing, onboarding, retention, market, store-listing analysis |
| Prompt rewrite | `prompt-optimize` | intake / handoff | improve, structure, translate, or rewrite prompts |
| Notion knowledge capture | `notion-page` | handoff | capture SOP, skill, prompt, checklist, project process, or Dev Kit lessons into Notion |
| App promotion planning | `app-market-promotion` | plan / handoff | post-launch promotion, channel selection, launch posts, store-link feedback, anti-spam handling |
| Frontend visual design | `frontend-design` | plan / execute / review | UI, layout, responsive behavior, visual polish, dashboards, landing pages |
| Frontend implementation | `frontend-developer` | execute / review | React components, state, client-side interaction |
| Browser verification | `playwright` or `browser-use` | verify | screenshots, viewport checks, click flows, rendered UI inspection |
| shadcn/ui components | `shadcn-ui` | execute / review | project already uses shadcn/ui or user asks for it |
| Current framework docs | `context7` | plan / execute | library, framework, SDK, API, CLI, or cloud-service usage |
| Project agent rules | `write-agents-md` | architecture (auto-triggered) | generate or update AGENTS.md after architecture is defined |
| UI-only polish | `ui-polish` | plan / execute / review | layout, spacing, hierarchy, responsive behavior, visual states of existing screens |
| Documentation update | `doc-update` | execute / handoff | update project docs, TODOs, PRDs, release notes, decisions, or summaries |
| Feature optimization | `feature-polish` | plan / execute / review | UX, state handling, refresh, validation, layout, feedback for existing working features |

## Plan Field

Use this field only when the task needs extra specialist rules:

```yaml
capability_skills:
  - frontend-design
  - playwright
```

If no specialist skill is needed, write:

```yaml
capability_skills: []
```

## Frontend Example

For a UI polish task:

```plain text
document_budget: none
execution_mode: simple-sequential
capability_skills:
- frontend-design
verification:
- browser screenshot
- desktop and mobile viewport check when layout can change
```

`frontend-design` provides design execution rules. The Nodeflow chain stays:

```plain text
nodeflow-request-router
-> nodeflow-plan
-> nodeflow-execute
-> nodeflow-review
-> nodeflow-verify
-> nodeflow-handoff
```
