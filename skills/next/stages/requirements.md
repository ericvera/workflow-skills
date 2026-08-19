# Write Requirements

Produce `requirements.md` in `<mise-directory>/` (from the config) from the approved goals (and mocks, when present).

## Sources

- `goals.md` — intent, scope, and the clarifying Q&A folded in at the goals gate.
- `mocks.html` + `mocks.context.md` (`full` route) — every logged tweak is a decision the requirements must reflect.
- The codebase — existing functionality the feature integrates with or must preserve.

An existing `requirements.md` (a reopened stage) is revised against the current sources, never regenerated.

## Writing the document

- **User-facing behavior only** — WHAT the system does, never HOW it's built.
- `REQ-<CATEGORY>-<N>` identifiers, grouped by functional area.
- MUST/SHOULD/MAY language; every requirement testable.
- Reference existing behavior that must be preserved.
- **Out of Scope**: behavior explored but deferred — mock variants that won't ship belong here.
- **Assumptions**: every non-obvious inference made where the goals and mocks were silent (edge cases, error handling, defaults).

```markdown
# Requirements

This document specifies the user-facing requirements for <feature>.

## 1. <First Major Area>

- **REQ-XXX-1:** The system MUST <requirement>.
- **REQ-XXX-2:** The system SHOULD <requirement>.

## Out of Scope

- <deferred behavior>

## Assumptions

- <non-obvious inference and the default chosen>
```

## Critic gate

Critic gate per `../references/interaction.md`: kind `requirements`, upstream `goals.md` (+ `mocks.html` / `mocks.context.md` when present).
