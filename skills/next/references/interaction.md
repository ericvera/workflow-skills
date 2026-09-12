# Shared interaction conventions

This file owns the conversation side of every stage: how gates run and how questions are asked. State semantics (approval hashes, the cascade, task completion) are the state engine's; its reports say what a stage needs.

## Gates

There are exactly two **human gates**: the goals(+mock) gate at the start and the acceptance pass at the end. Every stage between them self-approves through the critic gate and continues in the same session; the user is never asked to approve requirements or a plan.

### The human gate (goals stage only)

1. After presenting the artifacts (and any revisions), ask explicitly: "Approve goals<and mocks, when present>, or what should change?"
2. **Feedback, questions, or silence are NOT approval** — never advance on an unapproved artifact. Revise and re-present until the user says yes.
3. On explicit approval, run `node ../scripts/state.ts approve <mise-directory> goals route=<full|direct|bugfix>`, and on the `full` route `... approve <mise-directory> mock` after it. Decide the route before gating; it is recorded with the approval. A **changed re-approval** (new hash) also deletes every later stage's approval.
4. Commit the mise directory (artifacts plus `.workflow-state`): `mise: approve goals`.
5. Continue in-session: re-run the state engine report and dispatch the next stage without ending the turn.

### The friction log

`<mise-directory>/_friction.md` is the run's record of friction; only the retrospective role reads it. Wherever a rule says to log friction, append one line, `<stage or task>: <what happened>`, creating the file with a `# Friction` heading if missing, and include it in the checkpoint commit that follows.

While work is in flight, log every user chat message correcting the workflow's output or approach:

```
correction: <what>
```

Goals-gate iteration on the artifacts is not logged (the artifacts carry it); an acceptance flag is logged once, as its own `acceptance: user flagged <item> — <why>` line.

### The critic gate (requirements and plan stages)

After writing the artifact, dispatch `../roles/critic.md` (general-purpose Agent, synchronous; role `critic` per Model routing) with this prompt, all paths absolute:

```
Read and follow the instructions at <skill-dir>/roles/critic.md.
Artifact: <absolute path>   Kind: requirements | plan
Upstream: <absolute paths of the docs it must be checked against>
Mise config: <absolute path>
```

Then:

1. **Zero blocking findings = pass**; a fresh critic always finds _something_. Apply worthwhile minor findings without re-dispatching.
2. Blockers → revise and re-dispatch a fresh critic. Classify each blocker against the blocking findings of _all_ prior rounds, not just the last: the same underlying defect, however reworded, is **recurring**; anything else is **fresh**. Counts never decide.
3. All fresh → revise and re-dispatch.
4. Each recurring blocker is one **recurrence event**; log every event, the stalling one included:

   ```
   critic <stage>: blocker recurred (<short defect name>), sighting <n>
   ```

   The loop's **first** event earns that blocker one more fix attempt and a re-dispatch, unless a stall trigger fires; triggers take precedence.

5. **Stall** on exactly three triggers: `third sighting` (a blocker survives two fix attempts); `ping-pong` (a second recurrence event, on a different blocker); `budget exhausted` (the last allowed round still reports blockers, and revising there would ship a fix no critic saw).
6. **Budget: 5** rounds per artifact version, never scaled by task count or size; **ceiling: 8** across resets. A substantial mid-loop rewrite (a scope change replacing the artifact's content, not a large revision answering findings) resets the budget, the recurrence memory, and the event count. A user-directed round after a stall may exceed either budget and inherits the loop's memory.
7. At a stall, log friction, then stop and present the artifact, the per-round blocking counts, each remaining blocker labeled **recurring** or **fresh**, and the trigger that fired:

   ```
   critic <stage>: stalled (<trigger>) after <N> rounds
   ```

8. On a pass that took more than one round, log friction:

   ```
   critic <stage>: <N> rounds, blocking <count per round>
   ```

9. Then run `node ../scripts/state.ts approve <mise-directory> <stage>`, commit `mise: approve <stage>`, and continue in-session; `cleared_done` from a plan approval → mention the cleared IDs in the checkpoint report.

### Assumptions instead of questions

Stages past the goals gate never ask clarifying questions. Infer defaults from the goals, mocks, codebase, and `.claude/mise-config.md`, and record every non-obvious inference in an **Assumptions** section of the artifact. Stop and ask only on a genuine blocker: a contradiction between docs, or missing information no reasonable default resolves.

## Model routing

Every subagent has a named **role**: `implementer` (tasks, the `stuck` retry, per-task fixes, gate repairs, acceptance-blocker fixes), `reviewer` (per-task review), `critic` (the critic gate), `acceptance` (the acceptance pass), `explore` (plan-stage research), `documenter` (prose passes on tasks, per-task fixes, gate repairs, acceptance-blocker fixes), `retrospective` (the close-out retrospective). When the config's `## Models` section assigns a role a model, pass it as the Agent call's `model` parameter; no section, no entry, or the value `session` → pass no `model` parameter and inherit the session model. If the harness's Agent tool has no model override, dispatch without one; a model assignment never blocks a dispatch.

## Asking the user questions

Ask **1–3 related questions per turn**. Number the questions (`1.`, `2.`), letter the choices (`a.`, `b.`), one per line, so the user can reply `1a,2c`.

```
1. Where should X live?
   - a. Inside foo.ts
   - b. As a new module bar.ts
   - c. Inline in the caller

2. Should we add tests?
   - a. Yes
   - b. No
```

End every set with a recommendation line; if you are torn, still pick one and say so.

```
Recommend: 1a, 2a, 3c (reply `rec` to take all)
```

Add a brief reason after any pick that isn't obvious (`1a — keeps the change scoped to the existing module`); skip rationale otherwise. The line always ends with the `rec` hint: that reply accepts every recommended pick (a bare `a` is an option letter, never this shortcut).

Number any other list the user might pick from or refer back to, so they can respond by number.
