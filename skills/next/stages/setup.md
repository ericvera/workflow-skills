# Set up mise

Interview the user and write `.claude/mise-config.md` per `../references/config-reference.md` — read it first, and never copy it into the project. An existing config is revisited section by section with the user (current value shown, offer to keep).

## Interview

Ask in small batches per the question format in `../references/interaction.md`, inferring suggestions from the repo (package.json scripts, test directories, UI framework). Wherever a guide doc (`docs/`, top-level `*.md`) or project skill (`.claude/skills/`) already covers a section, offer a pointer to it (`See docs/testing.md`) as the default answer. Except for the required values and item 2, skipping an item writes no section or line.

1. **Required values** — the mise directory (suggest `.mise/`), the branch convention (infer from `git branch --sort=-committerdate`, else `feat/<slug>` / `fix/<slug>`), and the quality commands Format, Check, and Unit tests. Then optional **Task tests**: a scoped command with a `<path>` placeholder, inferred from package.json (`yarn test <path>`, `vitest run <path>`).
2. **Checklist** — the review checklist file's path (suggest `.claude/mise-checklist.md`). Create it with exactly this content when it does not exist:

   ```markdown
   # Review checklist

   Answer every rule pass / fail / n-a against your diff before committing.

   ## Code

   1. No leftover debug code — grep the diff for print/console/debugger statements and commented-out code.
   2. Every new file has a colocated test, or the task cites a Test exception — list the new files.
   3. No `REQ-*` identifiers in code, comments, tests, or strings — grep the diff.
   4. Nothing changed outside the task's Files to modify/create (the progress log excepted) — compare the diff's file list against the task.
   5. Changed behavior has a test that fails without the change — name it.

   ## Prose

   6. Every comment explains why or a non-obvious what; none restates the code — read each added comment.
   ```

   Tell the user it is theirs to edit: one rule per line as `<rule> — <how to check>`, `## Code` then `## Prose`, at most 15, and the retrospective proposes edits to it from their corrections.

3. **Mocks** — UI projects only: Mock conditions (e.g. "anything affecting Vue components"). Given any conditions, also Mock guidance.
4. **Test conventions**.
5. **Test exceptions** — work that should get no regression test, each with its substitute verification. Suggest the default `Purely visual changes (spacing, colors, cursor styles) — verify with before/after screenshots`, plus, with no e2e testing at all, a blanket `Anything that would need an e2e test (no e2e infrastructure exists) — verify with unit tests plus manual verification`.
6. **Skills & guides** — skills and guide docs agents should use. Prompt for an e2e test runner (`required` if e2e tests must never be run directly), an e2e writing skill, a manual-testing/screenshot skill, and any design, voice, or domain guides.
7. **Database migrations** — the migration generation command, if any.
8. **Backlog** — does the user track work externally (Todoist, Linear, Jira, a markdown file)? If yes, capture how to fetch the top items — tool, query, sections or labels — as freeform prose.
9. **Review notes** — does the user review diffs in an external notes tool (e.g. Delta Review)? If yes, capture how to read the branch's notes as freeform prose.
10. **Models** — which of the seven roles run on a model other than the session's? Suggest `implementer: opus`, `explore: opus`, `retrospective: opus`, with the gates (`critic`, `reviewer`, `acceptance`) on the session model. Ask **documenter** as its own question — should a separate documenter pass write all comments, JSDoc, and markdown docs after the code is done, and on which model? No → single pass, no line; the session model → write `documenter: session` explicitly, the one `session` entry the config keeps; `fable` / `opus` / `sonnet` / `haiku` → that value.
11. **Retrospective** — on by default; ask whether to keep it and write `Retrospective: off` only on a no.
12. **Ship** — `pr`, `merge` into the default branch (ask the style — squash / merge commit / rebase — and record it: `Ship: merge (squash)`), or `off`.

Code conventions the user offers belong in `CLAUDE.md`, not the config — offer to add them there.

## Write and verify

Write `.claude/mise-config.md` in the reference's generated-file shape, with only the sections the user filled. Print it and confirm it looks right.

Then ensure `CLAUDE.md` carries the shipping guard with the mise directory substituted (creating `CLAUDE.md` if needed, updating a previous mise guard line rather than adding a second):

> Never open a pull request for, or merge into another branch, any branch whose tree contains `<mise-directory>/` — that work is still in flight; run `/mise:next` on that branch to finish acceptance and cleanup first.

Once confirmed, commit the config, the checklist file, and the guard together: `mise: setup`.
