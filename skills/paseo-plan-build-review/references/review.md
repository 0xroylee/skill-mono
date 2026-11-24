# Freeze, independent review, corrections

Read this when a candidate is ready for review.

## Freeze

Stop all writers. Record repository, workspace, base commit, and every staged, unstaged and untracked change; preserve unrelated user edits. Bind a content identity to the whole candidate (matching `HEAD` alone does not cover uncommitted content):

```sh
{ git diff <base> --binary; git ls-files --others --exclude-standard | LC_ALL=C sort | while IFS= read -r f; do printf '%s  %s\n' "$(shasum -a 256 "$f" | cut -d' ' -f1)" "$f"; done; } | shasum -a 256
```

Keep a copy of the candidate (`tracked.diff`, `untracked/`, `status`, `sha256`) under `candidates/<unit>-<n>/` so later reviews can diff candidates and the commit can be checked against the reviewed bytes.

## Review assignment: required sections

Start a fresh reviewer on the selected review route through the launch gate. Include these assignment sections in order:

1. **Frozen candidate** — worktree, branch, base and HEAD, `git status --short` at freeze, identity and the exact pipeline to recompute it at start and before the verdict; a mismatch means `VERDICT: INSUFFICIENT_EVIDENCE`.
2. **Inputs** — the unit's assignment (scope, ownership, acceptance criteria), the accepted plan's `## Module design`, the implementer's report labelled as claims, the decisions document, repository rules, prior reviews of this unit and the previous candidate's location, and coordinator evidence (commands run outside the sandbox, with logs).
3. **Purpose and severity bar** — one paragraph, present from the **first** review: what the change is for, who it protects against, and therefore what counts as must-fix (wrong, behaviour-changing, out of ownership, breaks a repository rule, or a committed and CI-buildable escape of the thing being enforced), should-fix, and note (forms that cannot exist in committed buildable code, obfuscation, documented exclusions). Without this, an xhigh reviewer asked to probe will produce a new edge case every round.
4. **What to review** — ownership and scope; behaviour (moved files byte-identical apart from specifiers; edited logic read before and after; idempotency keys, transaction boundaries, lock order, error modes); acceptance criteria re-checked by command; design rules; module design (every new or changed interface, seam and adapter in the diff appears in the accepted Module design — an unplanned interface, a pass-through module or a one-adapter seam is should-fix, must-fix when it changes behaviour; a reviewer without the `mattpocock-skills:codebase-design` skill reads its `SKILL.md` under `~/.claude/plugins/cache/claude-plugins-official/mattpocock-skills/<highest-version>/skills/engineering/codebase-design/`); repository text accuracy; then the unit's specifics. Ask for verification of specific claims, not for "more angles" in general.
5. **Rules and checks** — copy `safety-boundaries.md` §4 (review only, no candidate edits, identity recomputed at start and end, one verdict), then the explicit list of commands the reviewer can run.
6. **Checkpoint and report** — report path, structure (token, identities, findings by severity with `path:line`, verification per area, commands with outcomes, limits), and exactly one final `VERDICT: PASS`, `VERDICT: NEEDS_CHANGES`, or `VERDICT: INSUFFICIENT_EVIDENCE`.

For a second or later candidate, add what changed since the previous one and what the coordinator decided on the previous review's should-fix items, so the reviewer judges those decisions instead of re-deriving the unit. When a stacked or sibling unit has known open findings, name them and say they are out of scope for this candidate.

## UI changes

Use the same reviewer and the candidate and screenshot rules in `../../pr-review-with-screenshots/SKILL.md`; this skill's verified route takes precedence over that skill's dispatch instructions.

- If a PR represents the complete frozen candidate, provide its URL and verified head. A known PR that cannot be verified is a blocker.
- Otherwise do a labelled local UI review; do not create or push a PR solely for this gate.
- Genuine screenshots only, preserved outside tracked source, inline with absolute paths, route/state and viewport.
- Missing required evidence yields `INSUFFICIENT_EVIDENCE` unless a confirmed defect warrants `NEEDS_CHANGES`.

## Corrections

- The initial build is round 0. A `NEEDS_CHANGES` verdict or a confirmed failed acceptance check can start correction 1 or 2. Record the round and attempt token before dispatch. Freeze the result of each correction and obtain a fresh independent verdict when a candidate exists. A replacement worker completing the same assignment keeps its round number.
- `PASS` on any reviewed candidate, including round 2, completes the unit when the acceptance checks pass. `INSUFFICIENT_EVIDENCE` calls for the missing evidence and a fresh verdict on the same frozen candidate. Evidence collection and infrastructure recovery do not consume a fix round.
- After round 2, any remaining confirmed blocker triggers the sub-issue handoff below. Trigger it earlier if consecutive reviews identify the same defect class. An unresolved failed acceptance check counts even if the final implementer could not produce a reviewable candidate. Distinguish a confirmed product failure from a worker or tooling outage.
- Record a user-requested change to accepted behavior as a scope revision. Preserve the round count for outstanding findings against the original criteria.
- Before committing a passed candidate, recompute the identity and compare the commit's tracked diff and new files with the frozen copy byte for byte. Pipe the pre-commit hook's output through `tail`; lint-staged can rewrite files, which the comparison catches.

## Failed-unit sub-issue and replan

1. Stop the failed unit. Preserve the candidate, verdict or failed check, attempted fixes and remaining findings. Never dispatch a third correction on that unit.
2. Use the parent tracking issue from the run record or task. Record a stable marker for the failed unit (run ID and unit ID) before the tracker write. On resume, look up the saved child or search the parent and repository for that marker before creating anything.
3. Create one sub-issue under that parent. Include the marker, expected and observed behavior, failed checks, both attempted fixes if applicable, remaining findings and acceptance criteria. Use a native parent/child relationship. If the tracker requires separate create and attach operations, save the child ID immediately, attach it, then read back the parent relationship. Recover a partial write by reusing that child.
4. Send the failed unit's evidence to a fresh planner on the selected Plan route. Require a revised approach, a reason the previous fixes failed, bounded work and acceptance checks. Accept the plan under `plan.md` and save it in the run record. Record the plan in the child issue body. On resume, reuse any accepted plan and verify the saved body before retrying an update.
5. If the parent is ambiguous, writes are restricted or the tracker is unavailable, save the issue body locally and proceed with the replan. Record the pending create, attach or plan update and the exact blocker. Resume from that operation. Never report the handoff complete until the link and accepted plan are verified.
6. Leave the parent unit unresolved. The handoff ends with a planned child. Implementing that plan is a separately scoped unit; do not restart the original fix loop or recursively dispatch children.

## Reading a review report

Pull findings by severity with bounded `sed -n` on the "Findings" section rather than dumping the whole report. Record in the run record: verdict, identity unchanged or not, each must-fix in one line, what was confirmed correct (so the next assignment can scope the reviewer), and the correction count.
