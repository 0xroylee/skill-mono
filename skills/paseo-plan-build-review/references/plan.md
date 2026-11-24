# Plan

Read this when writing a planner assignment or accepting a plan.

The planner makes no repository edits. Its assignment includes the user's objective, scope, existing constraints, and a unique report path. Require a concrete plan with relevant files, assumptions, risks, acceptance criteria, any independent units with explicit ownership, and a Module design section. For each unit, name the implementation approach, its acceptance check and any open correctness question. Assess the report against the request and resolve material gaps before accepting the plan. Planner acceptance does not expand authorization.

Use `--mode plan` for Claude. For Pi/Astra, send the same read-only assignment without Claude mode flags. If the planner cannot write the report path, request the report between `=====BEGIN <KIND> REPORT <token>=====` / `=====END <KIND> REPORT <token>=====` in the final message and save it from the session.

## Module design (required for every plan)

Every planner uses the `codebase-design` skill before planning (`mattpocock-skills:codebase-design` in Claude Code). If a Pi planner lacks that command, give it the installed skill file. Every plan carries a `## Module design` section:

- **Modules** — each module the change creates or changes, with its interface: everything a caller must know (signature, invariants, ordering, error modes).
- **Depth** — the deletion test for each new module: the complexity that would reappear across callers if it were deleted.
- **Seams** — where each interface lives. A new seam names its two adapters (for example production and test); with one adapter, the plan places no seam.
- **Reuse** — the existing modules and real toolchain the change builds on. A design that asks the real toolchain beats one that re-models it (a checker that asks the real resolvers beats one that models resolution; the latter never stops producing edge cases).

A change that creates or changes no module interface (copy, styles, config values, a fix inside one function body) writes `Module design: N/A — <reason>`.

Accept a plan only with this section present and checked against the same vocabulary: a shallow module, a pass-through that fails the deletion test, a one-adapter seam, or an N/A that the planned files contradict is a material gap. Send a plan with a material gap to a fresh planner with the gap named.
