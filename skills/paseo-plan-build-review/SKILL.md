---
name: paseo-plan-build-review
description: Coordinate planning, fast development and independent review through Paseo, including sub-issues and replanning for unresolved fixes.
---

# Paseo plan, build and review

Invoke `$paseo-plan-build-review` in Codex, `/paseo-plan-build-review` in Claude Code, or `/skill:paseo-plan-build-review` in Pi. Paseo manages the workers and workspaces.

This file is the router. It holds the roles, the invariants and the loop. Each stage has a reference under `references/`; read a reference when you reach its stage, not before.

## Roles

| Role | Paseo provider | Exact model | Thinking |
| --- | --- | --- | --- |
| Plan | `claude` by default, or `pi` for Astra | `claude-fable-5-1` or `openai-codex/gpt-6.2-astra` | `high` |
| Develop and fix | `codex` with Fast enabled | `gpt-6.1-terra` | `medium` by default, `high` for unresolved correctness questions |
| Develop a mechanical edit | `codex` with Fast enabled | `gpt-6-luna` | `medium` when the plan names the edit, existing pattern and acceptance check |
| Independent review | `pi` | `openai-codex/gpt-6.2-sol` by default, or `openai-codex/gpt-6.2-astra` for consequential trade-offs | `high` by default, `xhigh` for broad or unusually complex work |

An explicit user route takes precedence, including the existing opt-in Claude Sonnet route (`claude-sonnet-5-5`, `high`). Use Fast for every development and correction assignment unless the user explicitly selects another speed. If the selected provider cannot verify Fast, report the conflict before releasing the assignment. Select the coder for each accepted unit and retain its route for fixes. Pi's `openai-codex` prefix selects the subscription. An `openai/` API route needs an explicit user choice.

## Invariants

Boundary text in this workflow follows ASD-STE100 (`references/ste-rules.md`). The full rules are in `references/safety-boundaries.md`; copy its sections into assignments, do not paraphrase them.

1. Use the model and thinking that the user selected. If a route is not available, stop that assignment and report. Do not substitute a model, a harness, a thinking level or a billing route.
2. Start a fresh agent for every role and every attempt. Verify the route from runtime evidence before you send the assignment.
3. Run units in parallel only if their files are disjoint and their workspaces are isolated. Run import codemods one at a time.
4. The coordinator does not change a candidate. Three exceptions exist (`safety-boundaries.md` §2). Record each one and name it to the next reviewer.
5. Set token and time budgets for every unit. Allow at most two correction rounds after the initial implementation.
6. A candidate that changes after its freeze loses its verdict. Advance a stage only on the current attempt's report and evidence.
7. WARNING: Do not merge into a shared branch. Do not deploy. Do not call a live system. These need the user's separate and explicit words, each time. A commit, a push or a pull request also needs the user's words.

## The loop

1. **Kickoff** — resolve the Paseo CLI and routes, record authorization, create the run record, set per-unit budgets and the exhaustion action, list the commands the coordinator itself will run. Read `references/kickoff.md`.
2. **Launch** — preflight with a token generated beforehand, verify the route from runtime evidence, send the assignment from a file. Read `references/launch-gate.md`.
3. **Plan** — the planner uses `codebase-design` and writes a plan with a `## Module design` section or a reason it does not apply. Read `references/plan.md`.
4. **Implement** — implementers get a bounded unit, ownership, acceptance criteria, authorized checks and a report contract. Read `references/implement.md`.
5. **Freeze and review** — bind a content identity, start a fresh reviewer with the purpose and severity bar stated, get exactly one verdict. Read `references/review.md`.
6. **Correct or replan** — use `references/review.md` for round counting, failed reviews and the linked sub-issue handoff.
7. **Recover** — on timeouts, errors or long sessions read `references/recovery.md` before retrying.
8. **Finish** — accept only from the current attempt's evidence, archive owned idle workers, report. Token accounting and the report shape are in `references/finish.md`.

## Safety and boundary references

- `references/safety-boundaries.md` — the glossary and the numbered rules for authorization, the coordinator, the implementer, the reviewer, the candidate, budgets and publication. Read §1 and §2 at kickoff, §3 when you write an implementer assignment, §4 when you write a review assignment, §7 before you publish.
- `references/ste-rules.md` — how to write or change any boundary text: words, sentence limits, WARNING / CAUTION / NOTE form, and a checklist.

## Where things live

- Run records: `~/.local/state/paseo-plan-build-review/<run>/` (`run-record.md` first; `assign/`, `reports/`, `reviews/`, `candidates/`, `bin/`).
- Paseo CLI on this Mac: `/Applications/Paseo.app/Contents/Resources/bin/paseo`; `PASEO` in the references means that executable.
- UI changes: the reviewer also follows `../pr-review-with-screenshots/SKILL.md` for candidate and screenshot rules; this skill's verified route and model take precedence over that skill's dispatch instructions.
