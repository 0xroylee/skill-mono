# Kickoff: discovery, run record, budgets, coordinator limits

Read this once at the start of a run and again when a new unit is added.

## Discover and resume

Resolve the Paseo CLI once: the execution host's configured CLI or `paseo` on PATH; on this Mac the bundled fallback is `/Applications/Paseo.app/Contents/Resources/bin/paseo`. Preserve the selected daemon (`--host` or `--home`, when applicable) across every command. Do not switch to a different daemon after a connection failure.

Read installed `run`, `send`, `inspect`, `wait`, and workspace help. Run `PASEO provider diagnostic <provider> --json` and `PASEO provider models <provider> --thinking --json` for each selected provider. When Fast is selected, check the development model's Fast capability and verify the actual turn at launch. For Pi roles, also require `pi auth check --provider openai-codex --model <bare GPT model ID> --json` to report ready on the same host. If login is needed, ask the user to complete it without exposing credentials.

Use the actual caller workspace when Paseo provides it, or resolve the requested directory against `PASEO workspace ls --json`. Pass its exact workspace ID to every launch. For work on the main code, create a worktree workspace: `PASEO workspace create --isolation worktree --mode branch-off --base <ref> --new-branch <name>` (lands under `~/.paseo/worktrees/`); fresh worktrees need `bun install --frozen-lockfile`. Do not invent `PASEO_AGENT_ID`, copy another agent's identity, or assume a local directory exists on a remote daemon.

Before creating anything, inspect the run record and recorded agents. Resume completed stages from accepted reports; inspect unknown launch or send outcomes before retrying so uncertainty cannot create a duplicate writer.

## Run record

Keep it outside tracked source under `~/.local/state/paseo-plan-build-review/<run>/run-record.md`. Record: objective and the exact words that authorize each action class (implement, commit, push, PR, issue writes; merges and deploys are never implied), workspace and baseline per unit, accepted plan, each role/attempt's agent ID with requested and observed route, checkpoint and report paths, stage, frozen candidate identity, verdict, recovery count, correction count, and the budget table below.

## Budgets and the exhaustion action (set before the first implementer starts)

For every unit write a line like:

```
unit <key>: corrections ≤ 2 | tokens ≤ <N>M (implement + review) | wall clock ≤ <H>h
on token/time exhaustion: A or B | parent tracking issue: <repo and number, if known>
```

- The correction limit in `review.md` is fixed. Record tokens and wall time separately. The default token estimate is about 10× the first implementation's usage. Mark the estimate provisional until actual usage is available.
- If a token or time budget runs out first, use the following action. Default to B unless the user authorized A. A failed final correction follows `review.md` instead.
  - **A** — commit the last reviewed candidate as it is, open a **draft** PR whose body states the verdict and lists the open findings with locations and a suggested direction, post the same on the tracking issue, stop the unit.
  - **B** — commit nothing further, report the verdict and findings to the user, stop the unit.
- Change a token or time budget only when the user names the unit and the new value. Record the change. A larger resource budget does not buy another correction.
- **Mid-run signal.** When any unit reaches its second correction, or passes half of its token budget, write one status line in the chat before continuing with other units, so an absent user sees where the run is stuck when they return.

## What the coordinator can do itself

The rules are in `safety-boundaries.md` §1 and §2. Read them at kickoff and quote the user's authorizing words in the run record.

At kickoff, list the commands that the coordinator will run on a candidate because the sandbox cannot run them. Typical entries: `bun install` (network), a production build, a disposable-Postgres suite, one CI rerun per cause (`gh run rerun <id> --failed`). If a command is not on the list, ask the user before you run it.

## Parallel units

Parallel work needs disjoint files, isolated worktrees from the same recorded baseline, and one owner per shared file. Start every ready unit before waiting. When two units touch adjacent lines, finish both, then stack the later one on the earlier one with a merge commit (coordinator exception 2) and say "stacked on #N" in the PR. A guard whose baseline is keyed by file path is assembled last, on top of every file-moving unit.

## Kickoff questions for the user (one grilling round)

1. Record any explicit model, effort or speed override. Otherwise use the role table.
2. Record any token/time budget override and the parent tracking issue from the task or run record.
3. Record publish permissions already given. This workflow includes creating and linking the failure sub-issue and recording its replan. Honor an explicit restriction on tracker writes. Other issue writes, commits, pushes and pull requests need their own authorization.
4. Which checks the coordinator may run outside the sandbox.
5. Whether a mid-run status line in chat is wanted, and at which trigger.
