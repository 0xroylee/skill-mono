# Safety and boundary rules

This file is the single source for every boundary statement in this workflow. Copy the relevant section into the Workspace, Ownership or Rules section of an assignment. Do not paraphrase it there. The text follows `ste-rules.md`.

## Glossary

| Term | Meaning |
| --- | --- |
| user | the person who invoked the workflow and owns the repository decisions |
| coordinator | the agent that runs this workflow and dispatches workers |
| worker | a planner, an implementer or a reviewer started by the coordinator |
| candidate | the working tree of one unit, compared with its base commit |
| frozen candidate | a candidate with a recorded content identity, under review |
| unit | one bounded piece of work with its own worktree, budget and owner |
| run record | the file `run-record.md` in the run directory |
| shared branch | `integration`, `develop`, `main`, or any branch that other people build on |
| live system | production, staging, a payment provider, an email provider, a database that is not disposable |

## 1. Authorization

1. The user's words set what the coordinator can do. The run record quotes those words.
2. The coordinator can implement, commit, push and open a pull request only if the user named that action.
3. WARNING: Do not merge into a shared branch. Do not deploy. Do not call a live system. These actions need the user's separate and explicit words, each time. "ok" is not such a word.
4. The coordinator can create and link the failure sub-issue and record its replan. Honor the user's restrictions on tracker writes. Other issue or pull request writes require the user's authorization.
5. A plan that the planner accepted does not add authorization. A review that passed does not add authorization.

## 2. The coordinator

1. The coordinator does not change a candidate. Three exceptions exist; see item 2.
2. The coordinator can make only these three changes:
   - one line of build or tool configuration that a check needs;
   - the conflicting hunks of a merge that stacks one reviewed branch on another;
   - one line of documentation that a passed review named as a note.
   Record each change in the run record. Name each change to the next reviewer.
3. Every other change, including a one-word correction, goes to a fresh implementer.
4. The coordinator runs outside the sandbox only the commands listed at kickoff. If a new command is needed, ask the user first.
5. CAUTION: Do not send an assignment to a worker before its route is verified. Do not send a second assignment to a worker that has not finished.
6. Do not read a file that matches `.env*`, except `.env.sample`. Do not print a secret.
7. Do not grant a new permission to a worker to finish a stage. Report the permission request to the user.
8. Stop only the agents and servers that this run started. Do not delete a workspace, an unrelated agent, or the daemon.

## 3. The implementer

1. Change only the files that the assignment lists under Ownership.
2. Do not run a git command that writes (`commit`, `push`, `stash`, `checkout`, `reset`, `merge`, `rebase`).
3. Do not use the network. If a step needs the network, skip the step, finish the rest, and say so in the report.
4. Do not open a local port.
5. Do not write outside the worktree.
6. Do not start another agent.
7. Do not read a file that matches `.env*`, except `.env.sample`.
8. Do not change runtime behaviour unless the assignment names the behaviour change. Keep idempotency keys, transaction boundaries, lock order and error modes as they are.
9. Do not add an entry to a shrink-only baseline. If a new finding appears, list the finding and stop with `STATUS: FAILURE`.
10. End the final message with the report between the markers and the status line. The status line names the attempt token.

## 4. The reviewer

1. Review only. Do not change the candidate. Do not create a file inside the repository.
2. Do not commit, stash, push, or edit a GitHub issue or pull request.
3. Do not start another agent.
4. Write only the checkpoint file and the report file named in the assignment. Both are outside the repository.
5. Recompute the identity of the frozen candidate at the start and before the verdict. If a value differs, stop and report `VERDICT: INSUFFICIENT_EVIDENCE`.
6. Run only the commands that the assignment lists. Read-only commands are always permitted.
7. WARNING: Do not run a deployment. Do not call a live system.
8. Do not write into the repository with a command, except build output under `.next/`.
9. For experiments, copy files to a directory outside the repository.
10. Give exactly one verdict line at the end of the report.

## 5. The candidate

1. A candidate has one writer at a time.
2. CAUTION: A candidate that changes after its freeze loses its verdict. Freeze again and review again.
3. Before a commit, recompute the identity and compare the commit with the frozen copy byte for byte.
4. Keep edits that the user made and that do not belong to the unit. Do not revert them. Do not commit them.
5. Do not delete or overwrite a file that the unit does not own.

## 6. Budgets

1. Every unit has a budget line in the run record before its first implementer starts.
2. The user can change the token or time budget for a named unit. Each unit permits at most two corrections.
3. CAUTION: Stop implementation when the token or time budget is spent. Use the resource exhaustion action unless the failed unit already qualifies for the sub-issue handoff.
4. If blocking findings remain after two corrections, stop the unit. Open the linked sub-issue and replan under `review.md`. Use the same handoff earlier if two consecutive reviews identify the same defect class.

## 7. Publication

1. A commit message and a pull request body are in English. They end with the attribution lines that the user's instructions require.
2. A pull request body states the review verdict as it is. If the verdict is not `PASS`, the pull request is a draft, and the body lists the open findings.
3. A pull request body says which checks ran, who ran them, and what nobody verified.
4. WARNING: Do not push to a shared branch. Push only to the unit's own branch.
5. Do not rerun a CI job more than once for the same cause.
