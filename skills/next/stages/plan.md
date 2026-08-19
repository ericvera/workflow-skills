# Write Implementation Plan

Create a plan a developer with zero project context can follow, each task a self-contained file the implementer reads alone.

## Input

Working directory: `<mise-directory>/` (from the config).

**Revising a plan mid-execution**: completed task files sit in `implementation_plan/done/`, their work already in the repo (`_progress.md` and the git log). Plan the _remaining_ work from that reality. Approving the revision clears `done/`, so every task in the new plan is pending.

## Bugfix route

The state's `route` is `bugfix` → generate the plan from this fixed template. Read `goals.md` and the config, then write `implementation_plan/00_overview.md` plus:

- **No matching Test exception** — two tasks:
  - `01_01_regression_test.md`: write the regression test where `goals.md` records it, exercising the exact repro and asserting the _correct_ behavior, then **verify it FAILS for the bug's reason** — passing means it misses the bug (revise the assertions and retry), an unrelated failure is a setup problem to fix first. Copy the config's matching Skills & guides entries into its Guides; commit the failing test.
  - `01_02_fix.md`: the minimal fix that makes that test pass, naming the test file in its `Tests:` line. **The test file is frozen — modifying it is not allowed**.
- **A Test exception matched** (recorded in `goals.md`) — one task, `01_01_fix.md`: fix the bug and verify it the exception's stated way (e.g. before/after screenshots through the covering Skills & guides entry), evidence in the task report.

Fill each Background from `goals.md`: repro steps, expected/actual, affected code paths named concretely. Then the critic gate below.

## Feature routes

1. **Gather sources**: `goals.md`, `requirements.md` (the `REQ-*` IDs you must trace), and `mocks.html` / `mocks.context.md` if present.

2. **Analyze the codebase deeply** before writing: the patterns in play (state, routing, API endpoints, components, testing) and the exact integration points.
   - **Delegate broad exploration** to read-only subagents (role `explore`), one focused question each ("how does routing work"), asking each for a compact summary: key paths and their roles, relevant functions/types/variables, patterns with `file:line`, integration points.
   - Read directly only the files you cite most, and spot-check every `file:line` a subagent hands you before it goes into a task file.

3. **Create `implementation_plan/`** and write `implementation_plan/_exploration_notes.md`: the subagent summaries plus what you read yourself. Re-read it instead of source if context runs low.

4. **Design in the overview.** `00_overview.md` carries the design. Build only what the requirements need.

5. **Write** `00_overview.md` first, then each task file, sequentially and yourself, no subagents.

6. **Critic gate** per `../references/interaction.md`: artifact `implementation_plan/` (the overview plus task files), kind `plan`, upstream `requirements.md` (`goals.md` on the bugfix route).

## Output structure

Task files are `<phase>_<seq>_<task_name>.md` (`01_01_add_route.md`), numbered in execution order.

### `00_overview.md`

```markdown
# Implementation Plan

## Summary

<2-3 sentences: what is being built and why>

## Design

<How the pieces fit: components and their interactions (ASCII diagrams where
they help), data-model / schema / API-type changes, migration strategy, removal
plan if replacing existing code — a paragraph for a contained change, all of it
for a structural one.>

## Assumptions

<Non-obvious inferences made where the docs were silent, and the default chosen>

## Phases

- **Phase 1: <Name>** — <what this phase achieves>

## Phase Rationale

<why this order — what depends on what, what unblocks testing early>

## End-of-plan gate

<The pre-existing e2e and sanity suites the gate runs after the last task, by
name, and the Skills & guides entry that runs them — or "none" and why.>

## Task Index

| File              | Task                | Phase | Requirements         |
| ----------------- | ------------------- | ----- | -------------------- |
| `01_01_<name>.md` | <short description> | 1     | REQ-XXX-1, REQ-XXX-2 |
```

### Task file

```markdown
# Task X.Y: <Task Name>

## Goal

<what this task accomplishes>

## Requirements addressed

REQ-XXX-1, REQ-XXX-2

## Background

<Everything needed with zero project context: the feature in 1-2 sentences;
what prior tasks produced, named concretely ("Task 1.2 added `FooService` at
`path/to/fooService.ts`, registered in `container.ts:45`"); the patterns
involved, by file, function, type, variable; overview design decisions that
affect this task.>

## Files to modify/create

- `path/to/file.ts` — <what changes and why>

## Guides

<Skills & guides entries from the config whose conditions match this task,
verbatim, `required` flags preserved. Omit the section if none match.>

## Implementation details

1. <steps naming specific functions, types, patterns>

## Testing suggestions

- <how to verify this task works>
- <the unit tests covering the changed paths and any test this task writes, by
  file and name — or the Test exception and its substitute verification>

## Gotchas

- <things that look right but aren't>

## Verification checklist

- [ ] <task-specific checks>
- [ ] Tests: <unit tests to run / tests this task writes / or the cited Test exception and its substitute verification>
```

## Key rules for task files

- **Redundancy is intentional.** Repeat shared context rather than "see the overview" or "as in Task 1.1".
- **Name concrete code.** Not "follow the existing pattern" — "follow `SettingsPage.tsx` where `handleSettingChange` calls `updateField(...)` on line 213".
- **State what prior tasks produced** by file, type, and function — never "depends on Phase 2".
- **Route the guides.** Copy every config Skills & guides entry whose condition matches the task into its Guides section; the implementer and reviewer act on what the task file holds.
- **Include validation in every file.** Every task's verification checklist names the unit tests to run, the tests the task itself writes, and any cited Test exception's substitute verification — never a pre-existing e2e or sanity suite, which the overview's `## End-of-plan gate` owns, and no generic quality commands, which the implementer runs.

## Design principles

- **Task-size guardrail.** Files to modify/create lists **at most 5 source files**: tests, snapshots, and fixtures don't count, every other file does (markdown included), and workflow artifacts such as the progress log are never listed. Split tasks to fit. A task that must exceed the cap — a mechanical rename, a generated-file regeneration — ends its Task Index row's Task cell with ` — over cap: <one-line why>`; the critic treats an unjustified breach as blocking.
- **Every task ends green.** A plan needing a red intermediate state (one task breaks the build, a later one fixes it) is a planning bug: use expand-contract, or bundle the breaking change with its caller fixes. No green ordering → rethink the task boundaries.
- **Thin vertical slices over horizontal layers** — each phase produces working, testable functionality end-to-end.
- **Remove before building** — schedule removal of replaced code early.
- **Earlier phases unblock later ones** — foundations first, enabling incremental testing.
- **Test as you go** — verification in every task, not a final "test everything" phase.
- **E2e tests are mandatory for user-facing functionality** (new UI, new workflows, changed user-facing behavior) unless a config Test exception matches. Tasks write and run the new e2e tests the feature needs. If testability needs new test attributes, add them in the implementation tasks and say so.

## Database migrations

Tasks use the migration generation command from the config's Database migrations section, never raw migration tools.

## Do not

- Write code or code snippets in the plan (describe, don't implement)
- Include time estimates
- Create tasks smaller than meaningful progress
- Omit e2e coverage for user-facing functionality without a matching, cited Test exception
