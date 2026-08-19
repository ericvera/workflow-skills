# Goals (and Mock)

The pipeline's single human-gated stage.

## Input

The working directory is `<mise-directory>/` (from the config) and its `goals.md` already holds the user's description. Goals or mock artifacts that already exist (a reopened stage) are revised with the user, never regenerated.

## Bugfix route

Taken when the dispatcher classified the work as a bug fix or the recorded route is `bugfix` — a bug-understanding conversation, not the feature path below.

1. Read the relevant source and trace the code path to model the expected behavior. Reproduce the bug yourself when that helps and a Skills & guides entry covers manual testing.
2. Vague description → ask (batched): exact repro steps, expected vs actual behavior, error messages, intermittent or consistent.
3. Check the description against the config's **Test exceptions** — note which entry matched.
4. Rewrite `goals.md` as the durable record: repro steps, expected behavior (this becomes the test assertion), actual buggy behavior, the matched test exception (if any), and — when a regression test will be written — its planned file location per the config's Test conventions, with one line on why there.
5. **Gate** per `../references/interaction.md`, route `bugfix`: present the summary and have the user confirm the understanding and the test location. No code, no test, no plan before approval.

## Feature routes

### 1. Critique

Review `goals.md` critically for contradictions, missing considerations, technical or practical complications ahead, unstated assumptions worth validating, vagueness inviting scope creep, alternatives or enhancements worth considering.

The critique is working notes, never output: each finding surfaces as a clarifying question in the round below, carrying enough context to answer it, or as a direct edit to `goals.md` when no user call is needed.

### 2. One batched round of questions

The last point in the pipeline where a human is guaranteed present. Fold into one round (question format per `../references/interaction.md`):

- The critique points that need the user's call.
- What the requirements stage would otherwise ask: edge cases and error scenarios, user-facing behaviors not explicitly stated, integration points with existing functionality, what is explicitly OUT of scope.
- When a mock is coming (route below): where the feature surfaces (new page, modal, inline), which scenarios and states matter, what data it shows. Do NOT ask about visual detail — spacing, wording, layout.

Fold the answers into `goals.md`.

### 3. Route

Match the feature against the config's **Mock conditions**: any match → `full`; none, or no section → `direct`. "Mock this" / "no mock" from the user overrides at any point. Ask only if genuinely ambiguous — never guess silently, never ask when the conditions decide it or the state already records a route.

### 4. Mock (`full` route only)

Create `mocks.html` and `mocks.context.md` in the mise directory.

- **Match the product.** When the config has Mock guidance, follow it: read the UI code at its UI code root and closely approximate the real look and feel — never guess. Apply matching Skills & guides entries (voice guides, design guides).
- In the single HTML file, mock each relevant scenario and state inside a realistic application window with realistic sample data, each labeled with a unique ID (`1`, `2`; variants `1A`, `1B`) and a one-line description of what it demonstrates. Offer variants of the same idea.
- **Fit the user's mental model, not the implementation.** Audit every concept the mock introduces: genuinely new, worth its learning cost, or internals leaking into the UI? Prefer concepts the product already teaches; list the survivors under New Concepts in `mocks.context.md` — an unlisted concept becomes an assumption instead of a user decision.
- **Cover the canonical states, not just the happy path**: empty (no data yet), loading when meaningful, error including where errors appear, and every submit or action's success and failure outcomes.
- Initialize `mocks.context.md`:

  ```markdown
  # Mock Context

  ## Original Description

  <the user's original description>

  ## Clarifying Q&A

  <the questions asked above and the user's answers>

  ## New Concepts

  <each concept the mock introduces that the product doesn't already teach, with why no existing concept fits — "None" when only familiar concepts are used>

  ## UI Tweaks Log

  <every piece of mock feedback: what was requested, what changed>
  ```

### 5. Iterate and gate

Present goals (and mocks) together and iterate on both in one loop: mock feedback into the UI Tweaks Log (note _(Logged: …)_ when you record one), goal edits straight into `goals.md`.

Then the human gate per `../references/interaction.md`, recording the route decided above (`full` / `direct`).
