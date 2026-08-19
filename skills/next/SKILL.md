---
name: next
description: |
  Runs the next step in the mise workflow. Use when the user reports a
  bug or defect to fix, asks what to work on next, or wants to start or
  continue work on a feature.
argument-hint: "[setup | what to work on]"
disable-model-invocation: true
allowed-tools: Bash(node ${CLAUDE_SKILL_DIR}/scripts/state.ts *) Bash(git add *) Bash(git commit *) Bash(git mv *) Bash(git branch *) Bash(git switch *) Bash(mkdir *)
---

# Next

You are the **dispatcher**: read the state, continue or start the work, and route to the right instruction file — never do stage work except under a dispatched stage's instructions.

Copy this checklist and check off each step:

```
Progress:
- [ ] 1. Match the arguments to a mode
- [ ] 2. Load context, check the config gate
- [ ] 3. Run the state engine, then continue or start the work
- [ ] 4. Act on next_action: announce, then dispatch; after a self-approving stage, loop from step 3
```

## Modes

- Empty `$ARGUMENTS` → run the checklist end to end.
- `setup` → dispatch setup even when the config is filled, then stop.
- `?`, alone or followed by anything → print `Usage: /mise:next [setup | <what to work on>]` and stop.
- Anything else → a work description: carry it into Continue or start the work below.

## Load context

Read `${CLAUDE_SKILL_DIR}/references/interaction.md` (question format, gate rules, friction rules — `next` obeys them like every stage) and the project's `.claude/mise-config.md` (its Mise directory value is `<mise-directory>` below).

Ground rules:

- To **dispatch** a stage or setup is to read its instruction file and follow it. Dispatched files assume the two files above are loaded.
- This skill's directory is `${CLAUDE_SKILL_DIR}`; relative paths inside a dispatched file resolve against that file, never against the project.
- Every `.workflow-state` read and write goes through the **state engine** below; never inspect or edit a state file, or apply its rules, by hand.
- Commit the mise directory at every checkpoint — creation, approvals, recorded decisions, completed tasks, cleanup — so any checkout of the branch resumes the work; checkpoint subjects take the bare `mise:` prefix (`mise: approve goals`).

**Config gate** — dispatch setup to fill the blocked value, then resume where the gate fired:

- No config file, or no Mise directory value (the directory itself is legitimately absent between pieces of work) → setup, then resume with the original `$ARGUMENTS`.
- Missing **Format**, **Check**, or **Unit tests** command, or a `Checklist:` value missing or naming a file that does not exist (setup re-creates the starter file) → blocks the execute dispatch alone, re-firing on `stage:execute` and on `acceptance`.
- Missing **Branch convention** → blocks starting new work alone.

Nothing else blocks.

## The state engine

`<mise-directory>/.workflow-state` drives every routing decision:

```
node ${CLAUDE_SKILL_DIR}/scripts/state.ts report <mise-directory> --write
```

- The engine needs Node 24+; a missing or older `node` → report it and stop.
- One run initializes the state file on a fresh start and computes `next_action`.
- Re-run it only after mise-directory content is created, moved, or deleted, or a stage records an approval.
- `reopened: [...]` means a doc changed after approval — surface which stages reopened and why before acting.
- A broken or hand-edited state file gets an `{error}` naming the user's options: relay it and stop.

## Continue or start the work

A mise directory with any content in it means **work is in flight** — one piece at a time per branch.

**In-flight gate** — act on the report's `in_flight` field:

- **`true`**, no description → continue that work; done here.
- **`true`**, any description (bug reports included) → report and stop, starting nothing: "Work is already in flight on this branch — run `/mise:next` (no arguments) to continue it. Finish it before starting new work; for a bug fix or another feature, use a fresh branch."
- **`false`**, no description → the **backlog prompt**; treat the answer as a description and re-enter this gate:
  - the config has a `## Backlog` section → follow it to fetch the top to-do items and present them numbered — "Pick a number, or describe what you want to work on:".
  - no `## Backlog` section → ask "Describe what you want to work on:".
- **`false`**, a description → start it:
  1. **Classify** it:
     - clearly a defect — "fix", "bug", "regression", "crashes" → the bugfix route.
     - clearly new behavior → a feature.
     - ambiguous → ask "Bug fix or new feature?"; never guess silently.
  2. **Ensure a work branch**, from `git branch --show-current`:
     - `main` or `master` → derive a name from the config's Branch convention and the description (the classification picks the pattern when the convention names one for fixes) and `git switch -c <name>`.
     - any other branch with no commits past the default branch (`git log main..HEAD` empty, or `master..HEAD`) and a name that follows the Branch convention for this work → use it, no question.
     - any other branch otherwise → ask "Use the current branch `<branch>` for this work, or create a new one from it?" — switch only if the user chooses new.
  3. **Start it**: create `<mise-directory>/`, write the description verbatim to `goals.md`, commit. Carry the classification into the goals dispatch.

Continuing with `goals.md` missing → ask "What's the goal for this work?" and write the answer verbatim.

## Act on `next_action`

Announce the move, then dispatch its file:

```
Running stage: <stage>
```

| `next_action`                              | File                                         |
| ------------------------------------------ | -------------------------------------------- |
| `stage:goals`, `stage:mock`                | `${CLAUDE_SKILL_DIR}/stages/goals.md`        |
| `stage:requirements`                       | `${CLAUDE_SKILL_DIR}/stages/requirements.md` |
| `stage:plan`                               | `${CLAUDE_SKILL_DIR}/stages/plan.md`         |
| `stage:execute`, `acceptance`, `close_out` | `${CLAUDE_SKILL_DIR}/stages/execute.md`      |
| the config gate, or the `setup` argument   | `${CLAUDE_SKILL_DIR}/stages/setup.md`        |

- **`stage:<name>`** → artifact existence is never approval.
- **`acceptance`** → when execute returns, offer the backlog prompt.

## Stopping rules

Once a stage is dispatched, stop only at the goals gate, the acceptance pass, or a real blocker; when the next step is unambiguous, resolve and run it rather than asking to confirm. When a dispatched stage self-approves at its critic gate (requirements, plan), do not end the turn or tell the user to re-run `/mise:next` — re-run `report --write` and dispatch the next stage in the same session.

## Do not

- Advance past the goals(+mock) gate without the user's explicit approval, or past a critic gate without a critic pass.
- Re-ask decisions already recorded in `.workflow-state`.
- Stop between self-approving stages.
- Touch `.workflow-state` except through `${CLAUDE_SKILL_DIR}/scripts/state.ts` — `next`'s only state write is `report --write`.
- Delete the mise directory unless the user has confirmed the work finished or abandoned — the acceptance-confirmed cleanup is the only unprompted deletion.
