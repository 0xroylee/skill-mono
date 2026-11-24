# Finish: acceptance, publication, accounting, report

Read this when a unit or the run ends.

## Accept and publish

Accept completion only from the current attempt's report and evidence. Record the reviewed candidate identity and the verdict before archiving owned, idle workers (`PASEO archive <id>`). Stop only agents or servers this run started; keep workers when asked; preserve reports and screenshots. Do not delete workspaces, unrelated agents or the daemon.

Publication follows the authorization recorded at kickoff. Commit messages and PR bodies are English; the PR body states the review outcome as it is (`PASS`, or the exhaustion action's wording with the open findings), the checks run and by whom, what nobody verified, merge order for stacked branches, and any manual checks owed after landing. Never present a draft published under exhaustion action A as reviewed-and-passed.

CI: a step cancelled at the job timeout with the test runner silent is the known bun-test hang (`gh run rerun <id> --failed` once); a job cancelled after 15 minutes with zero steps and the annotation "The job was not acquired by Runner of type hosted" is a GitHub runner outage (check githubstatus.com, rerun once later, do not loop).

## Token accounting

Report usage by role and by unit from the provider session files; subscription routes have no invoice, so give tokens, not money (Pi's `cost` field is an estimate at API list prices).

- **Codex**: rollouts under `~/.codex/sessions/YYYY/MM/DD/rollout-*.jsonl`, moved to `~/.codex/archived_sessions/` when the agent is archived. First line `session_meta.payload.cwd` identifies the worktree; the last `event_msg` with `payload.type == "token_count"` holds `info.total_token_usage` (`input_tokens`, `cached_input_tokens`, `output_tokens`, `reasoning_output_tokens`, `total_tokens`), cumulative for the session. `turn_context.payload.model` distinguishes the main model from `codex-auto-review` subagents.
- **Pi**: `~/.pi/agent/sessions/--<cwd slug>--/*.jsonl`; sum `message.usage` (`input`, `output`, `cacheRead`, `cacheWrite`, `totalTokens`, `cost.total`) over assistant messages; `model_change` gives the model.
- **Claude coordinator**: `~/.claude/projects/<cwd slug>/<session>.jsonl`; for `type == "assistant"` records take the largest `message.usage` snapshot per `message.id` (`input_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`) and sum; split by timestamp to isolate one run inside a longer session.

Expect cached input to dominate (about 95% on the 2026-10-06 run); report it separately from fresh input and output.

## Final report shape

1. Per unit: review verdict, correction count, CI state and any authorized PR. For unresolved work include the sub-issue URL, parent link and replan status. Identify a pending tracker operation explicitly.
2. What changed, as a stack diagram and a before/after tree of moved paths.
3. Checklist of the plan's stages with status and the hard gates reached.
4. Token usage by role and by unit; wall-clock span.
5. Deviations from this skill's defaults, each with cause and the rule that now covers it (or the proposal if none does).
6. What nobody verified, and the decisions owed by the user.
