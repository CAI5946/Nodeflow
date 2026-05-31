# Nodeflow Skills

Nodeflow is a risk-adaptive workflow skill system for coding agents.

It provides one visible entry skill, `nodeflow`, plus focused `nodeflow-*` nodes for planning, execution, review, verification, release preparation, and handoff. The goal is to help agents choose the smallest useful process for each task instead of forcing every request through the same fixed workflow.

## Contents

- `.codex-plugin/plugin.json`: Codex plugin manifest
- `skills/nodeflow/`: main orchestration skill and shared references
- `skills/nodeflow-*`: specialized workflow nodes
- `agents/openai.yaml`: optional OpenAI/Codex display metadata for each skill

## Install

Install this repository as a Codex plugin, or copy the skill directories manually.

For manual installation, copy `skills/nodeflow` and `skills/nodeflow-*` into your agent skills directory.

Example:

```powershell
$skillsRoot = "$HOME\AgentSkills"
Copy-Item -Recurse .\skills\nodeflow* $skillsRoot
```

Then restart or refresh your agent so it can rescan skills.

## Usage

Start with the `nodeflow` skill for most tasks. It will route to the smallest suitable chain based on task risk, scope, and required verification.

For direct use, invoke a specific node such as:

- `nodeflow-risk-assessment`
- `nodeflow-plan`
- `nodeflow-execute`
- `nodeflow-review`
- `nodeflow-verify`
- `nodeflow-handoff`

## Notes

Nodeflow is tool-agnostic. Some references mention optional capabilities such as browser automation, documentation lookup, calendar or notes tools. Treat those as optional integrations and use the closest available tools in your environment.

## License

MIT
