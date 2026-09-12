# mise — Claude Code workflow skills

A Claude Code plugin for building complex features autonomously. You describe the goal up front; requirements, planning, and execution run unattended, checked at every stage by fresh-context critic and reviewer agents.

The name comes from _mise en place_ — prep everything before the pan gets hot.

## Install

```
/plugin marketplace add https://github.com/ericvera/mise-claude-plugin
```

```
/plugin install mise@ericvera
```

Requires Node.js 24+ on your PATH — the [state engine](docs/state-machine.md) is TypeScript that Node [runs natively](https://nodejs.org/en/learn/typescript/run-natively), no build step.

### Update

```
/plugin update mise@ericvera
```

## Usage

One command drives everything; its first run in a project walks you through configuration.

| Command                       | What it does                                                 |
| ----------------------------- | ------------------------------------------------------------ |
| `/mise:next`                  | Run the next stage of the work in flight (or start new work) |
| `/mise:next some description` | Start a new feature or bug fix from that description         |
| `/mise:next setup`            | (Re)run project configuration                                |

All project-specific details live in the generated `.claude/mise-config.md` — Mise directory, Branch convention, the quality commands (Format, Check, Unit tests, and the scoped `Task tests:` slot), `Checklist:`, Mock conditions, Test exceptions, Skills & guides, `## Models`, `## Review notes`, `## Backlog` for your external to-do list, and `Ship`. `## Review notes` tells the retrospective how to read your external review notes; every reviewer turn there is a correction. Setup writes the checklist's starter file (default `.claude/mise-checklist.md`): `## Code` rules then `## Prose` rules, one per line as `<rule> — <how to check>`, at most 15, yours to edit.

## How it works

One piece of work per branch, driven entirely by `/mise:next` — new work gets a branch from your configured naming convention, or reuses the one you're on.

1. **Goals** _(human gate)_ — a conversation that critiques your goal, asks one batched round of questions, and iterates on an HTML mock when configured. You approve once.
2. **Requirements** — generated from the goals and mock, assumptions recorded explicitly, self-approved by a critic agent.
3. **Plan** — a fine-grained implementation plan with full-context task files, each held to the task-size guardrail of 5 source files (tests, snapshots, and fixtures don't count) or carrying a written justification; self-approved by a critic agent that blocks an unjustified breach.
4. **Execute** — each task runs in a fresh-context implementer subagent, verified with Format, Check, and its tests (the `Task tests:` slot scoped to what it touched when configured, else full `Unit tests`, plus the tests the task writes), then reviewed and committed. The implementer and every fix subagent answer your checklist against their own diff and record the answers in the commit message; the reviewer confirms each `pass` against the diff, answers for itself the rules left `n-a` or without confirmable evidence, and reports wrong answers as defects alongside correctness, task-spec, and guide failures. A `documenter` line under `## Models` turns on two-pass mode: implementers write code only, and a documenter pass writes the comments, JSDoc/docstrings, and markdown docs for each task's commit before the reviewer sees it, following your doc guides, committed with a `Docs:` subject; every later commit that changes code gets the same pass. The config's `## Models` section can delegate the generation roles (implementer, explore, documenter, retrospective) to cheaper models; the gates (reviewer, critic, acceptance) default to your session model.
5. **End-of-plan gate** — after the last task, full verification runs once: Format, Check, the full `Unit tests` suite, and the e2e and sanity runs the plan named.
6. **Acceptance** _(human gate)_ — one verdict per requirement, per done task whose commits lack its checklist answers, and per `## Prose` rule checked against the branch's `Docs:` commits; the gate's results stand, so e2e never re-runs. On your confirmation a retrospective mines the run for improvements to your guides, config, `CLAUDE.md`, or a new doc, and flags workflow flaws as plugin candidates. Every correction you made becomes a proposal too: a checklist rule added, sharpened, or merged, a durable doc, or — when the correction is about the workflow's own approach — a config or plugin-candidate proposal. Each is yours to adopt or reject. Then the working docs are cleaned up and the branch ships per your `Ship` value: `pr`, `merge`, or `off`.

Bug fixes take a shortened route: a bug-understanding conversation, then a fixed test-driven plan — write the failing regression test, then fix without touching it.

The same workflow by actor — USER acts only at the two human gates (the adopt/reject prompt rides the second):

```text
  USER                  |  ORCHESTRATOR                     |  SUBAGENTS
------------------------+-----------------------------------+--------------------
  describe the work ----|--> goals conversation (+ mock)    |
  [? approve goals? ] <-|--- present goals (+ mocks)        |
    feedback -----------|--> revise, present again (loop)   |
    explicit yes -------|--> approve goals, commit          |
                        |  - - - unattended from here - - - |
                        |  write requirements --------------|--> critic: severity-tagged defects
                        |  revise on blockers, commit <-----|---- (stall: recurring blockers / round budget)
                        |  write plan ----------------------|--> critic (same loop)
                        |                                   |
                        |  for each task: dispatch ---------|--> implementer, documenter (two-pass), reviewer
                        |                                   |      (defects --> fix subagent)
                        |  task done, commit (loop)         |
                        |  end-of-plan gate:                |
                        |    format, check, unit tests, e2e |
                        |  acceptance pass -----------------|--> acceptance: per-item verdicts
  [? confirmed? ] <-----|--- present the verdicts           |
    items wrong --------|--> fix, re-run gate + acceptance  |
    confirmed ----------|--> retrospective -----------------|--> retro: guidance + checklist proposals
  [? adopt which? ] <---|--- present proposals              |
    picks --------------|--> apply picked, commit           |
                        |  cleanup: delete mise dir         |
                        |  ship: PR / merge / off, done     |
------------------------+-----------------------------------+--------------------
```

**Caveat:** results are exactly as good as your verification — the workflow leans hard on linters, unit tests, and especially end-to-end coverage to keep long runs grounded.

## Design

mise targets work beyond a single prompt (trivial) or Claude Code's plan mode (moderate ambiguity): complex features that need externally imposed rigor. Three principles follow:

- **Optimize for autonomous runs** — the machinery exists so work can run overnight, unattended.
- **Force thinking at the correct level** — each stage focuses the agent on one layer: goals, requirements, design, implementation.
- **Create artifacts at each step** — they enable review, limit reliance on context, and give rewind checkpoints. Scaffolding, not output: committed as work progresses (any checkout resumes it), removed in a final cleanup commit that leaves the code, its tests, and the git history.

## Development

Load the plugin straight from a checkout:

```
claude --plugin-dir /path/to/workflow-skills
```

Follow the [skill authoring guide](docs/skill-authoring.md) when editing instruction files — stage files orchestrate; each fresh-context subagent role except explore has its own self-contained file under `skills/next/roles/`. The state engine's spec lives in [state-machine.md](docs/state-machine.md); its tests run with `node --test skills/next/scripts/state.test.ts`.

## Credits

Forked from [saeedn/workflow-skills](https://github.com/saeedn/workflow-skills) by Saeed Noursalehi. His original skills — and many conversations with him — heavily inspired this workflow's design.
