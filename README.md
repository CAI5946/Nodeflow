# Nodeflow

Nodeflow is a skills-only workflow plugin for coding agents.

It provides one visible entry skill, `nodeflow`, plus focused `nodeflow-*` nodes for planning, execution, review, verification, release preparation, and handoff. The goal is to help the agent choose the smallest useful process for each task instead of forcing every request through the same fixed workflow.

## Quickstart

For Codex:

```powershell
codex plugin marketplace add CAI5946/Nodeflow
codex plugin add nodeflow@nodeflow
```

Start a new agent thread after installation, then ask:

```text
Use $nodeflow to plan and execute this change.
```

## How It Works

Nodeflow is not a slash-command system, MCP server, or separate UI. It is a collection of skills.

Agents can load relevant skills from their descriptions. You can also name a skill directly with `$nodeflow` or a specific node such as `$nodeflow-review`.

## Installation

### Codex CLI

Register this repository as a plugin marketplace:

```powershell
codex plugin marketplace add CAI5946/Nodeflow
```

Install the plugin:

```powershell
codex plugin add nodeflow@nodeflow
```

Update later:

```powershell
codex plugin marketplace upgrade nodeflow
codex plugin add nodeflow@nodeflow
```

### Claude Code

Nodeflow includes a Claude plugin manifest:

```text
.claude-plugin/plugin.json
```

Use Claude Code's plugin installation flow for GitHub-hosted plugins, then start a new thread so Claude can discover the bundled skills.

### Cursor

Nodeflow includes a Cursor plugin manifest:

```text
.cursor-plugin/plugin.json
```

Use Cursor's plugin installation flow for GitHub-hosted plugins. The manifest points Cursor at the root `skills/` directory.

### Manual Skills Install

If your agent supports plain skill directories, copy the skill folders manually:

```powershell
$skillsRoot = "$HOME\AgentSkills"
Copy-Item -Recurse .\skills\nodeflow* $skillsRoot
```

Then restart or refresh your agent so it can rescan skills.

## Usage

Use the entry skill for most tasks:

```text
Use $nodeflow to plan and execute this change.
```

Use a specific node when you know what you need:

```text
Use $nodeflow-risk-assessment to classify this task before editing.
```

```text
Use $nodeflow-review to review the current diff.
```

```text
Use $nodeflow-verify before claiming this is done.
```

## Basic Workflow

1. `nodeflow` routes the request.
2. `nodeflow-risk-assessment` decides scope, risk, document budget, and verification needs.
3. `nodeflow-plan` creates a task plan only when the work needs one.
4. `nodeflow-execute` performs the work with the selected execution mode.
5. `nodeflow-review` checks alignment, regressions, and missing verification.
6. `nodeflow-verify` confirms results with command output or observable evidence.
7. `nodeflow-handoff` summarizes changes, risks, and next steps.

## What's Inside

- `skills/nodeflow/`: main entry skill and shared references
- `skills/nodeflow-*`: specialized workflow nodes
- `.codex-plugin/plugin.json`: Codex plugin manifest
- `.agents/plugins/marketplace.json`: Codex marketplace manifest
- `.claude-plugin/plugin.json`: Claude plugin manifest
- `.cursor-plugin/plugin.json`: Cursor plugin manifest

## Limitations

- Nodeflow is a skills-only plugin.
- It does not provide slash commands.
- It does not start an MCP server.
- Optional capability skills mentioned inside Nodeflow are not bundled unless they are part of your agent environment.

## More Docs

- [Codex installation and troubleshooting](docs/README.codex.md)

## License

MIT
