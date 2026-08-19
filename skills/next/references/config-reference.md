# mise-config reference

Reference for setup (`../stages/setup.md`), never copied into projects: setup
interviews the user and writes a minimal `.claude/mise-config.md` with only the
sections that apply — values and freeform sections, no instructional prose, since
every stage and every fresh-context subagent reads it.

Never restate consumption rules here or in the generated config — they belong to the
files that read the value.

## Generated file shape

```markdown
# Mise Configuration

Mise directory: .mise/
Branch convention: feat/<slug> for features, fix/<slug> for bug fixes
Checklist: .claude/mise-checklist.md
Ship: pr

## Quality commands

- Format: yarn format
- Check:
  - yarn lint
  - yarn typecheck
- Unit tests: yarn test
- Task tests: yarn test <path>

## Mock conditions

- Anything affecting Vue components

## Mock guidance

Product: Acme Dashboard. UI code root: `src/ui/`.

## Test conventions

See docs/testing.md

## Test exceptions

- Purely visual changes (spacing, colors, cursor styles) — verify with before/after screenshots

## Skills & guides

- run-e2e (skill, required): running end-to-end tests — never run them directly
- docs/voice-guide.md: any user-facing copy

## Models

- implementer: opus
- explore: opus
- documenter: fable
- retrospective: opus

## Database migrations

`yarn prisma migrate dev --name "<migration message>"`

## Backlog

Use the Todoist MCP to list tasks in the project "Engineering", sorted by priority.

## Review notes

Delta Review notes file for the branch, per the `delta:review-notes` skill's contract.
```

## Section reference

Required — the config is unfilled without these:

- **Mise directory** — where in-flight work lives (suggest `.mise/`): stage artifacts and `.workflow-state`, nothing else, so its existence means work is in flight.
- **Branch convention** — how work branches are named, one line using `<slug>` for a kebab-case slug of the work (e.g. `feat/<slug>` for features, `fix/<slug>` for bug fixes).
- **Checklist** — a top-level value line: the path of the project's review checklist file (suggest `.claude/mise-checklist.md`). The file: `# Review checklist`, one intro line, `## Code` then `## Prose`, rules numbered continuously across both, one per line as `<rule> — <how to check>`, each answerable pass / fail / n-a against a diff. Cap 15 rules, never blocking.
- **Quality commands** — Format, Check (lint + typecheck), Unit tests, plus optional Task tests. Each slot is one command or a list (nested bullets) run in order. `Task tests:` is scoped: a `<path>` placeholder (`yarn test <path>`) that per-task verification fills. These are _run_ commands only — how tests are written is Test conventions.

Optional — omit the section when it doesn't apply. A body is inline values or a
pointer to an existing project doc or skill (`See docs/testing.md`), which its reader
follows; prefer the pointer whenever the content already lives in the project:

- **Mock conditions** — bulleted conditions deciding _whether_ a feature gets an HTML mock before requirements: matching any → `route: full`, otherwise direct.
- **Mock guidance** — how mocks should look: product name, UI code root, look-and-feel notes.
- **Test conventions** — authoring: where each kind of test lives, naming, frameworks, fixtures.
- **Test exceptions** — bulleted `condition — alternative verification` entries: matching work is verified the stated way instead of by the regression-test and mandatory-e2e rules — the method changes, verification never disappears. Suggest the purely-visual → screenshots entry as a default.
- **Skills & guides** — one entry per line: `name-or-path (skill|doc[, required]): when to use`. `required` means it MUST be used whenever its condition matches (e.g. an e2e runner that must never be bypassed).
- **Models** — which model each subagent role runs on, one entry per line: `- <role>: <model>`. The seven roles are `implementer`, `reviewer`, `critic`, `acceptance`, `explore`, `documenter`, and `retrospective`. Values are the Agent tool's model identifiers — `haiku`, `sonnet`, `opus`, `fable` — plus `session`, meaning inherit the session model. Setup writes only non-`session` entries and omits the section when every role is `session`. One exception: a `documenter` line's presence switches two-pass mode on, so setup writes `documenter: session` explicitly when the user wants that pass on the session model.
- **Database migrations** — the migration generation command, if the project has one.
- **Backlog** — freeform instructions for fetching top to-do items from an external tracker; read verbatim.
- **Review notes** — freeform instructions for reading the user's external review notes for the branch (e.g. its Delta Review notes file, per the `delta:review-notes` contract); read verbatim, like Backlog.
- **Retrospective** — a top-level value line, only ever written as `Retrospective: off`: disables the post-acceptance retrospective, the pass that mines the finished run for guidance improvements. Omit the line to keep it on — the default.
- **Ship** — a top-level value line: `pr` | `merge (<style>)` | `off` — what the execute close-out does with the finished branch: open a pull request, merge into the default branch, or leave shipping to the user. The `merge` style — `squash`, `merge commit`, or `rebase` — is the user's setup choice, recorded in the value, never assumed. Omit the line → the close-out asks each time.
