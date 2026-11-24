# Implement

Read this when writing an implementer assignment.

## Implementer assignment

Give the implementer: the objective, the accepted plan text (including its Module design) or the bounded unit, the exact workspace and baseline (commit hash), ownership (the files it may change and nothing else), acceptance criteria, authorized checks, the report contract. Run checks only when the user requested them or the task's authorization covers them; do not add tests by default. The implementer builds the interfaces the Module design names; a needed interface outside it goes in the report as a design question.

Copy `safety-boundaries.md` §3 into the assignment's Rules section. Add the unit's own list of files that the implementer must not touch (sibling units' files, checker files that another unit owns).

Codex implementers run in a workspace-write sandbox: no network, no local ports, no git writes, no writes outside the worktree. Therefore:

- the report is delivered in the final message between `=====BEGIN IMPL REPORT <token>=====` / `=====END IMPL REPORT <token>=====` with a last line `STATUS: SUCCESS <token>` or `STATUS: FAILURE <token>`; extract it from the rollout (tolerate a truncated end marker);
- `bun install`, production builds, disposable-Postgres suites and anything needing sockets are coordinator-run (listed at kickoff); the implementer continues past them and says so in the report;
- the coordinator commits.

## Correction assignment

Use the initial development route, reasoning effort and speed for each correction. Give the fresh implementer the review report, blocking findings, acceptance checks and ownership. Use `review.md` for round counting and the transition to replanning.

## Accepting implementation

Read the report and the actual diff before accepting. For document-heavy candidates, before the first freeze check every `path:line` citation mechanically (path exists, range in file), re-run every recorded command against its stated number, and read the highest-stakes tables against source; implementers' citations are unreliable and each review round spent on them costs 20–45 minutes.

## Parallel units

Create isolated workspaces from the same recorded baseline, including relevant uncommitted inputs. Give shared files and integration one owner. Start all ready units before waiting; serialize when isolation or ownership cannot be established. Do not start from a remote default branch merely because it is the tool's default.

## Checkpoints

After each coherent work batch and before handoff or compaction, have the worker report the attempt token, role, workspace/baseline, evidence, diff identity, remaining tasks, decisions, checks and owned running commands. For a Codex worker confined to the worktree, collect the checkpoint from its session and have the coordinator save it outside tracked source. Workers with permission to write the checkpoint path can replace it atomically.
