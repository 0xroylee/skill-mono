---
name: pr-review-with-screenshots
description: Use when reviewing Paseo UI changes with screenshots after implementation finishes.
---

# PR review with screenshots

## When to use

Review completed UI changes in `packages/app` and related Electron UI.

Read `docs/qa.md`, `docs/design.md`, and `docs/browser-capture-harness.md` before reviewing.
Read the assignment, spec, accepted plan, previous findings, and implementer's report.
Treat the implementer's report as claims to verify.

When called by `paseo-plan-build-review`, follow its `references/review.md` and `references/safety-boundaries.md`.
Keep its authorization limits and outer report contract. This skill adds the screenshot review gate.

## Candidate rules

- Review one frozen candidate. Confirm all writers have stopped.
- Record repository, workspace, branch, base commit, HEAD, and staged, unstaged, and untracked changes.
- Record the whole candidate's content identity. HEAD alone does not identify uncommitted changes.
- Use the caller's identity procedure, including tracked changes and untracked file contents.
- Recompute the identity at the start and immediately before the verdict.
- If it differs, stop reviewing. Mark the candidate unverified and require a new freeze.
- Never edit, format, fix, revert, or otherwise change the candidate.
- Preserve unrelated user edits.
- Verify that the running app contains the frozen candidate. Record build provenance.
- If a PR represents the complete candidate, record its URL and verified head.
- Otherwise label the work a local UI review. An unverifiable known PR is a blocker.

## Screenshot capture

Build a capture checklist from the acceptance criteria and `docs/qa.md`.

- Cover every affected platform: iOS, Android, browser web, and Electron on macOS, Windows, and Linux.
- Record tested, unverified, or not affected for each platform, with reasons.
- Do not treat a narrow browser viewport as native-platform evidence.
- Capture compact and wide layouts, plus relevant breakpoint transitions.
- Capture light and dark appearances when the UI is themed.
- Capture default, populated, empty, loading, error, selected, hover, pressed, and disabled states where applicable.
- Include open menus, sheets, dialogs, scrolling, and keyboard-visible states affected by the change.
- Observe first load and slow data arrival. Capture layout changes before and after data resolves.
- Capture neighbouring UI that shares the changed components or layout.
- Capture before and after where possible, using the same data, viewport, theme, and state.
- Use an existing baseline or a separate authorized copy. Never switch or modify the frozen candidate.
- Mark unavailable baseline screenshots unverified.

Capture genuine pixels from the running app. Open and inspect every screenshot.
Keep full-view originals; label any supplementary crops or annotations.
Use video or interaction evidence for motion, focus, and performance claims that still images cannot prove.

For Electron browser capture, follow `docs/browser-capture-harness.md`.
Use real Electron evidence for guest compositor and keyboard-boundary changes.
Do not alter parking geometry or focus behavior to make screenshots pass.
The harness defaults to `packages/desktop/capture-harness/out/`; do not run it in the frozen repository.
Use an authorized external copy or coordinator-provided artifacts, and record provenance.

Save permitted artifacts outside the repository in the caller's assigned evidence directory.
For standalone reviews, use `/tmp/pr-review-with-screenshots/<candidate-id>/<attempt>/`.
Name screenshots `<before|after>-<platform>-<route>-<state>-<theme>-<width>x<height>.png`.
Record logical viewport size and device pixel ratio separately.

The caller's write limits still apply. If only checkpoint and report writes are permitted, have the coordinator supply screenshots.
Do not create screenshot files, start servers, or run capture commands beyond the assignment's permissions.
Mark missing required captures unverified.

## What to check

- Match the spec and acceptance criteria, including visible behavior after interaction.
- Check theme tokens in `packages/app/src/styles/theme.ts` and reuse of canonical primitives.
- Check typography, hierarchy, spacing, radii, colors, and control sizes against `docs/design.md`.
- Preserve established density and section rhythm.
- Align glyphs and content rails, not touch-target boxes.
- Check clipping, wrapping, overflow, safe areas, window controls, and overlay placement.
- Keep layout stable across loading, errors, badges, and changing counts.
- Check empty, hover, pressed, selected, and disabled presentation.
- Keep actions discoverable on native and compact layouts where hover is unavailable.
- Check compact navigation and wide pane layouts independently.
- Exercise neighbouring UI for regressions.
- Inspect relevant automated coverage; screenshots do not prove behavior or performance.
- For hot-path changes, require the before-and-after measurements specified by `docs/qa.md`.

## Evidence

Record candidate identities, capture provenance, platform coverage, route, state, theme, viewport, and reproduction steps.
Include commands with outcomes and raw output where available.
Embed screenshot evidence using absolute paths.

For each confirmed finding, cite a screenshot path and the relevant candidate `file:line`.
State expected behavior, observed behavior, user impact, and the required correction.
Rank findings using the caller's severity bar: must-fix, should-fix, then note.

List evidence gaps separately. Mark each unverified and explain what would verify it.
Never invent a screenshot, source location, test result, or platform result.
Do not infer runtime correctness from source inspection alone.

## Verdict

Return one UI gate result: `APPROVE` or `CHANGES REQUESTED`.
Put severity-ranked findings and unverified items before the final verdict line.

Use `APPROVE` only when the candidate identity is unchanged, required evidence is complete, and no blocking finding remains.
Otherwise use `CHANGES REQUESTED`. Distinguish confirmed defects from missing evidence.

The coordinator maps this gate into its outer report contract:
approval maps to `PASS`; confirmed blocking defects map to `NEEDS_CHANGES`;
identity mismatch or missing required evidence maps to `INSUFFICIENT_EVIDENCE`.
Do not emit a second workflow verdict in this UI report.

## Forbidden

- Do not edit the candidate or create files inside the repository.
- Do not run Git commands that change state.
- Do not commit, push, create or edit a PR, post comments, merge, or deploy.
- Do not restart the main Paseo daemon on port 6767.
- Do not run the full test suite.
- Do not start another agent or call a live system.
- Do not expand permissions to obtain evidence. Report the limitation.
