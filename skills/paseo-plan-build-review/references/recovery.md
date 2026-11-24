# Timeout and session recovery

Use the selected Paseo executable, daemon, workspace, and recorded agent ID throughout recovery. Preserve the accepted plan, authorization, completed stages, partial edits, and evidence. `PASEO` denotes the executable resolved by the parent skill.

## Classify before retrying

Read the run record, then inspect the recorded agent and a bounded slice of its timeline:

```text
PASEO inspect <agent_id> --json
PASEO logs <agent_id> --tail 20 --json
```

| Observation | Action |
| --- | --- |
| `wait` returns `timeout`; agent still runs | Continue waiting on the same ID in bounded intervals. Keep the assignment active. |
| Blocking `send` times out, or send/launch outcome is unknown | Reconcile the attempt token with the agent/timeline and run record before another mutation. A lost receipt does not prove that work was rejected. |
| Compaction, provider retry, reasoning, or a known tool is active | Allow the operation to finish. Ten minutes elapsed is a diagnostic checkpoint, not an automatic failure or session lifetime limit. |
| Permission, question, authentication, rate limit, or service outage | Record the blocker and use its authorized response or documented recovery path. Restarting a session does not resolve an external blocker. |
| Idle/exit/error without the current attempt's valid result | Keep the stage incomplete; inspect the checkpoint, actual files, and errors, then use the recovery steps below. |
| Current attempt has a valid result | Assess its evidence and candidate identity before accepting the stage. |

The installed Paseo blocking `send` may stop waiting after ten minutes; the main workflow uses `--no-wait` to avoid that limit. A timeout from waiting, transport, or Pi RPC can leave the underlying operation running. Keep one assignment and one writer until its actual state is established. If bounded logs omit the relevant operation, widen the slice around its recorded timestamp.

After ten minutes without observable progress, inspect for a pending request, compaction, provider retry, or known long operation. Silence alone cannot establish a crash. Continue when the operation is live; if it is demonstrably stuck, capture available state and retire the owned attempt before replacement. If liveness or exclusive writer ownership cannot be established, persist the blocked stage and report it.

## Recover a long Pi context

Let Pi's automatic compaction and internal provider recovery settle first. When a context-overflow or compaction failure leaves the role idle, preserve its checkpoint and current diff, and resolve any underlying authentication/provider problem.

For a usable idle session, try one manual compaction through Paseo:

```text
PASEO send <agent_id> --no-wait --json "/compact Preserve the objective, accepted plan, authorization, checkpoint path, completed work, remaining tasks, and candidate identity."
PASEO wait <agent_id> --timeout 60 --json
```

Inspect the timeline for errors and successful completion. A compaction event labeled completed or an idle state alone is insufficient: some adapters also emit a completed event when compaction fails. A timeout while compacting leaves its outcome unknown; reconcile it before issuing another compaction or continuation.

After successful compaction, reverify the effective route and send a bounded same-role continuation using the checkpoint, actual diff, acceptance criteria, and remaining tasks. Assign a unique recovery token/report path and require that token in the result. If compaction fails or the session remains unusable, use a fresh worker as below. Keep automatic compaction enabled; preserve the selected model, thinking, and billing route. Do not change global settings or arbitrary timeout values as recovery.

## Replace a failed or stuck worker

1. Preserve the last valid checkpoint and reconstruct missing facts from the accepted plan, current files/diff, and evidence. Mark uncertain work unverified. Do not require a crashed worker to produce a final checkpoint.
2. Stop only the owned agent with `PASEO stop <agent_id> --json`. Inspect/wait for the interrupted turn and its owned mutating commands to settle. Archive the retired agent with `PASEO archive <agent_id> --json` and confirm the archive. An interruption receipt alone does not establish that every writer or queued task is stopped. Confirm no owned mutating child process or pending work can continue on the candidate; otherwise block replacement. Preserve unrelated services and user changes.
3. Mark the old attempt superseded in the run record. Start a fresh agent through the parent's preflight and model verification gate, using the same selected route and intended workspace with the actual partial edits preserved. Give it a new attempt token, checkpoint/report paths, and the continuation packet: objective, authorization, accepted plan, ownership, baseline/current candidate, verified completed work, remaining tasks, evidence, and unresolved decisions. Resume unfinished work; reconcile partial edits before making new writes.
4. Accept only the replacement attempt's result. Preserve completed stages; old or late reports cannot settle the replacement. If the reviewer was replaced, the fresh reviewer must cover the entire frozen candidate and return its own verdict. Prior partial notes are evidence, not a PASS. A changed candidate requires a new freeze and review.

Allow at most two coordinator recovery attempts per failed role assignment by default, including manual compaction/continuation and fresh replacement. Ordinary wait timeouts and Pi's own internal retries do not consume this budget. Record the recovery count separately from the review correction count. After exhaustion, persist the incomplete stage, retained candidate/checkpoint, errors, and next required action; report the blocker instead of retrying indefinitely or advancing without evidence.
