# Execute Implementation Plan

## Role

You are the **orchestrator**: you dispatch every unit of work to a fresh-context subagent and implement, review, or document nothing yourself. Your context holds the overview, the TODO list, and the subagents' reports, never a task file or source. Each role is a general-purpose Agent, dispatched synchronously by the templates below with absolute paths; role model and friction lines per `../references/interaction.md`.

## Before starting

1. **Read the overview**, `<mise-directory>/implementation_plan/00_overview.md`.
2. **Get `tasks_done`** from the state report that dispatched you (or `node ../scripts/state.ts report <mise-directory>`): a task is done when its file sits in `implementation_plan/done/`.
3. **Baseline gate**, before the plan's first task only (`done/` empty): run the config's `Format`, `Check`, `Unit tests`; anything fails → stop and report.
4. **Build the TODO list** from the remaining Task Index rows, in order: `Implement <mise-directory>/implementation_plan/<filename>`.

**Two-pass mode** = the config's `## Models` has a `documenter` line; single pass otherwise. In two-pass mode:

- Every commit that changes code is followed, before anything else runs, by a documenter pass scoped to it — Per task step 3's prompt, with `Fix scope: <what was fixed>` in place of `Task file:` outside a task.
- A `prose` defect goes to the documenter, a `not-prose` one to an implementer.

## Per task

Work the TODO list in order, each entry in_progress while it runs:

1. **Dispatch an implementer**:

   ```
   Read and follow the instructions at <skill-dir>/roles/implementer.md.
   Task file: <this task's file>
   Progress log: <mise-directory>/implementation_plan/_progress.md
   Mise config: <project>/.claude/mise-config.md
   ```

2. **Failure** → log friction `task <ID>: <blocked|stuck> — <one-line cause>`, then:
   - `blocked` → stop and relay the report.
   - `stuck` → one fresh implementer, that prompt plus `Previous attempt's failure report: <the report>`; a second failure of either kind → stop and relay both.

3. **Success**, two-pass mode → **dispatch a documenter**:

   ```
   Read and follow the instructions at <skill-dir>/roles/documenter.md.
   Task file: <this task's file>
   Commits: <the implementer's commit>
   Progress log: <mise-directory>/implementation_plan/_progress.md
   Mise config: <project>/.claude/mise-config.md
   ```

   `Docs pass committed` → its commit joins the reviewer's `Commits:`; `Docs pass: nothing to document.` → nothing to add. A failure → as step 2, one fresh documenter on `stuck`; friction `task <ID>: documenter <stuck|blocked> — <cause>`.

4. **Dispatch a reviewer**, `Commits:` holding every commit this task produced:

   ```
   Read and follow the instructions at <skill-dir>/roles/reviewer.md.
   Task file: <this task's file>
   Commits: <hash(es)>
   Mise config: <project>/.claude/mise-config.md
   Progress log: <mise-directory>/implementation_plan/_progress.md
   ```

   `none` → step 5. Defects → log friction `task <ID>: review found <defects, one line>`, then one fix round:

   - Single pass → an implementer, step 1's prompt plus `Defects: <the list>`.
   - Two-pass → an implementer, step 1's prompt plus `Defects: <the not-prose defects>`, when there are any; then the documenter, step 3's prompt with the fix commit as `Commits:` and the `prose` defects as `Defects:`, whichever exist.

   Anything but a clean success report from it → stop and report.

5. **Record it done**: `mkdir -p <mise-directory>/implementation_plan/done && git mv <mise-directory>/implementation_plan/<task file> <mise-directory>/implementation_plan/done/`, commit `mise: task <ID> done`, mark the entry completed.

## End-of-plan gate

Run it when the last task completes in-session, on any dispatch with all tasks done and no acceptance recorded, and again from step 1 after any commit lands past a finished gate (a repair, an acceptance-blocker fix):

1. **`Format`**; a run leaving changes in the tree is a failure, and the repair commits them.
2. **`Check`**.
3. **`Unit tests`**, the full suite.
4. **The e2e and sanity runs** the overview's `## End-of-plan gate` section names, through the config's Skills & guides entry covering such runs, honoring `required`. No such section → grep the `End-to-end tests:` lines of the task files in `done/`, your one sanctioned peek, and run their union the same way.

**A failed step** → log friction `gate: <what failed>`, dispatch one repair, re-run the gate from step 1; a second failure → stop and report. The repair:

- Prose failure in two-pass mode (`Check` failing inside a comment or doc) → the documenter, Per task step 3's prompt with `Fix scope: <what to fix>` + `Defects: <what failed>` in place of `Task file:` and `Commits:`.
- Anything else → an implementer, Per task step 1's prompt with `Fix scope: <what to fix>` + `Defects: <what failed>` in place of `Task file:`; it commits `Fix:`.

## Acceptance and close-out

On a `close_out` dispatch, start at step 4's retrospective, skipping its recording clause; that done too (its adoption commit exists, or the user says so) → step 6.

1. **Dispatch acceptance** with the gate's results:

   ```
   Read and follow the instructions at <skill-dir>/roles/acceptance.md.
   Requirements: <requirements.md, or goals.md on the bugfix route>
   Overview: <mise-directory>/implementation_plan/00_overview.md
   Progress log: <mise-directory>/implementation_plan/_progress.md
   Mise config: <project>/.claude/mise-config.md
   Gate results: <the gate's summary>
   ```

2. **Present its verdicts as returned** and ask whether to close the feature out.
3. **Only items the user flags** are blockers: log friction `acceptance: user flagged <item> — <why>`, fix each as the gate's repair does (a prose item in two-pass mode through the documenter, anything else through an implementer, `Fix scope:` + `Defects:` naming the item), then re-run the gate from its step 1, and acceptance after it.
4. **On the user's confirmation**, record it (`node ../scripts/state.ts approve <mise-directory> acceptance`, commit `mise: accept`). Then the **retrospective**, unless the config carries `Retrospective: off` (→ step 6):

   ```
   Read and follow the instructions at <skill-dir>/roles/retrospective.md.
   Mise directory: <mise-directory>
   Mise config: <project>/.claude/mise-config.md
   ```

   Present its proposals verbatim with its `Recommend` line and ask which to adopt, by number; no proposals → say so and go to step 6. Adopting is never required.

5. **Apply the adopted proposals** exactly as proposed: project guidance only (the config, its `Checklist:` file, `CLAUDE.md`, a guide doc, a new doc plus its registration line), never source or plugin files, in one ordinary commit `Adopt retrospective learnings: <summary>`. Relay plugin candidates; never act on them.
6. **Clean up**: delete `<mise-directory>/` entirely, commit `mise: cleanup`, report the feature finished.
7. **Ship** per the config's `Ship` value: `pr` → push and open a pull request summarizing the work; `merge` → merge into the default branch in the recorded style (`merge (squash)`; none recorded → ask); `off` → report the branch ready. No `Ship` value → ask which of the three, and suggest `/mise:next setup` to record it. Shipping fails (auth, conflicts) → report the branch and the chosen action.

## Stopping rules

Stop only at: a failed baseline gate, a `blocked` report, a `stuck` report that survived its retry, a fix round without a clean success report, a second gate failure, an ambiguity the overview and config cannot resolve. A subagent reporting that a task file authorizes a red intermediate state is a planning bug: stop and ask the user to regenerate the plan. A task note never beats a skill default on a safety or verification rule.

## Do not

- Skip the baseline gate before the plan's first task, the end-of-plan gate, or the acceptance pass, or let a subagent skip verification
- Fix anything in the acceptance verdicts the user did not flag
- Retry past the bound: one fresh subagent per `stuck`, one fix round per task, one repair per gate run
- Delete the mise directory, open a PR, or merge before the user's confirmation and the cleanup commit
- Apply a retrospective proposal the user didn't adopt, or let one touch source code or the plugin's files
