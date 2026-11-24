# Launch gate: preflight, route verification, sending the assignment

Read this before launching any role.

## Token first

Generate the attempt token before the launch call (`<ROLE><n>-<8 hex>`), write it to the run directory, and use it in the preflight prompt. A placeholder in the preflight is a recorded deviation.

## Preflight

Every fresh role starts with a harmless preflight prompt: identify the role and the attempt token, require a brief `READY <token>` reply, and prohibit tools, repository work, implementation and delegation until the assignment arrives. This is an instruction, not enforced isolation. Launch in the background and save the returned agent ID immediately.

Pi and Claude roles:

```text
PASEO run --no-wait --json --workspace <workspace_id> --title "<run>/<role>/<attempt>" --provider claude --model <selected-Claude-model> --thinking <effort> --mode plan "<preflight prompt>"
PASEO run --no-wait --json --workspace <workspace_id> --title "<run>/<role>/<attempt>" --provider pi --model <selected-Pi-model> --thinking <effort> "<preflight prompt>"
```

Use the selected model from the role table. Do not give Pi Claude-specific modes. Preserve configured permissions. Save the launch receipt and extract the agent ID even when a workspace notice precedes the JSON.

For a Codex build or fix with Fast enabled, launch through Paseo `create_agent` with `provider: "codex/<selected-model>"`, the worktree's `workspaceId`, the preflight as `initialPrompt`, and `settings: {modeId: "auto-review", thinkingOptionId: "<selected-effort>", features: {service_tier: "priority"}}`. Read the installed provider's feature schema. If it exposes the older boolean for Fast, use `features: {fast_mode: true}`. Use the selected provider's equivalent if another fast-capable route was requested. Fast and reasoning effort are separate settings. For an explicit standard-speed override, request the provider's standard tier and verify it on the worker turn.

## Verify the route from runtime evidence

```text
PASEO wait <agent_id> --timeout 60 --json
PASEO inspect <agent_id> --json
PASEO logs <agent_id> --tail 20 --json
```

Confirm fresh identity, workspace, the `READY <token>` reply, provider, exact model and thinking. `inspect` exposes `Provider`, `Model`, `Thinking`, `Cwd`, `Status` (Paseo 0.10.3; use the current version's fields). Inspection alone can echo configured values, so add provider-session evidence:

- Pi: `~/.pi/agent/sessions/--<cwd with / replaced by ->--/*.jsonl` has `model_change` (`provider`, `modelId`) and `thinking_level_change` lines; find the file by grepping for the token.
- Codex: match the rollout `turn_context` model and effort to the worker. Match the daemon's turn-start `serviceTier` to the same agent/thread and turn. Require `priority` (or the provider's documented legacy `fast`) on a Fast assignment. Agent config and catalog availability alone do not prove that Fast ran. Rollouts live under `~/.codex/sessions/YYYY/MM/DD/` and move to `~/.codex/archived_sessions/` after archive.
- Claude: the session jsonl under `~/.claude/projects/<cwd slug>/` has `message.model` on assistant lines.

Unknown or mismatched explicit settings block release: stop the owned worker and report the gap.

## Send

Write the assignment to a UTF-8 file and use `PASEO send <agent_id> --prompt-file <absolute_path> --no-wait --json`. A send receipt proves enqueue; idle state or process exit alone does not prove acceptance.

## Wait

Prefer finish notifications. Otherwise poll with a background watcher that reads a `label<TAB>agent_id` list, inspects each agent every 30 s, and exits when any is idle, errored or has a pending permission; the harness re-invokes you when it exits. A wait timeout leaves the assignment active: keep the same agent and assignment, do not resend. A permission or question state needs an authorized response; never grant new permissions just to finish a stage.

Observed durations with fast mode on this Mac: bounded corrections 2–12 min, a 187-file codemod with build and tests about 25 min; Sol xhigh reviews 15–45 min.
