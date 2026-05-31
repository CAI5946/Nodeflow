# Installing Nodeflow on Codex

Nodeflow is distributed as a Codex plugin marketplace repository.

## Install

Register the marketplace:

```powershell
codex plugin marketplace add CAI5946/Nodeflow
```

Install the plugin:

```powershell
codex plugin add nodeflow@nodeflow
```

Start a new Codex thread so the newly installed skills are visible to the agent.

## Verify

List installed plugins:

```powershell
codex plugin list
```

Then ask Codex:

```text
Use $nodeflow to explain how Nodeflow would route a small bug fix.
```

Nodeflow is skills-only. You should not expect a slash command, MCP server, or standalone UI.

## Update

Refresh the Git marketplace snapshot:

```powershell
codex plugin marketplace upgrade nodeflow
```

Reinstall the plugin from the refreshed marketplace:

```powershell
codex plugin add nodeflow@nodeflow
```

Start a new Codex thread after updating.

## Manual Install

If you are not using Codex plugin marketplaces, copy the skill directories manually:

```powershell
$skillsRoot = "$HOME\AgentSkills"
Copy-Item -Recurse .\skills\nodeflow* $skillsRoot
```

Restart or refresh your agent after copying.

## Troubleshooting

If Codex cannot find the plugin:

1. Check that the marketplace is configured with `codex plugin marketplace list`.
2. Refresh marketplaces with `codex plugin marketplace upgrade`.
3. Reinstall with `codex plugin add nodeflow@nodeflow`.
4. Start a new Codex thread.

If Codex can see the plugin but does not use it automatically, mention the skill explicitly:

```text
Use $nodeflow for this task.
```

For narrower workflows, mention a specific node:

```text
Use $nodeflow-risk-assessment before changing files.
```

```text
Use $nodeflow-review to review the current diff.
```
