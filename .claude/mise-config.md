# Mise Configuration

Mise directory: .mise/
Branch convention: feat/<slug> for features, fix/<slug> for bug fixes
Ship: merge (squash)

## Quality commands

- Format: yarn format
- Check: yarn typecheck
- Unit tests: yarn test

## Test conventions

Tests live next to the code they cover as `*.test.ts`, using the Node built-in test runner (`node --test`).

## Test exceptions

- Changes to markdown that ships as guidance — skill/reference instruction files, `docs/`, and `README.md` — verify via critic review plus a dry-run walkthrough (instruction files) or a term-for-term consistency check against the file it documents (`README.md`, `docs/`); no unit test
- Anything that would need an e2e test (no e2e infrastructure exists) — verify with unit tests plus manual verification

## Skills & guides

- docs/skill-authoring.md (doc): when writing or editing skill instruction files
- docs/state-machine.md (doc): when touching state.ts or workflow-state semantics

## Models

- implementer: opus
- explore: opus
- retrospective: opus
