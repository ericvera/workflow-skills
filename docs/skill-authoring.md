# Skill authoring guide

The conventions this plugin's instruction files follow, and why. The sources below carry general guidance; this file records decisions.

_Guidance snapshot: August 2026. Re-read the sources for anything newer and fold it in._

## Writing style

- **Match specificity to fragility, not a house style.** The dial is "degrees of freedom": open-ended work gets heuristics, fragile order-dependent work explicit rules. This workflow is a gated state machine, so its files spell out every branch.
- **Rationale only where a rule would otherwise be misapplied.** A why-clause earns its place when a reader would otherwise apply it wrongly in an unanticipated case ("commit at every checkpoint — so a dead machine costs nothing"); elsewhere the bare rule is shorter and just as followed.
- **The removal test.** For each line ask "would removing this line cause a mistake?" — if not, cut it. Anything derivable from a stated contract goes: the config gate is stated once, not restated wherever it applies. Carry the smallest set of high-signal lines: rules get lost in the noise.
- **One rule per position.** A rule buried mid-parenthesis or as a fifth clause gets skipped under load; load-bearing rules get their own bullet.
- **One term per concept, and name the recurring ones.** "Mise directory", "stage", "config gate" mean the same thing in every file: a synonym reads as a new concept, a named contract is referenced in three words. Prefer plain English over coined terms.

## Structure

- **User-invoked only.** `next` starts a long, stateful process, so it carries `disable-model-invocation: true` and never preloads into the workflow's subagents. Its `description` is the user's `/` menu documentation, third person, what + when.
- **Harness substitutions over prose conventions.** `${CLAUDE_SKILL_DIR}`, `$ARGUMENTS`, and `${CLAUDE_PROJECT_DIR}` are substituted into the SKILL.md body at invocation, so anchor commands on them rather than restating a path-resolution rule per command. Substitution never reaches dispatched files.
- **Role statement first.** One or two sentences of identity and what the executor never does ("You are the dispatcher … never do stage work yourself").
- **Role files for fresh-context subagents.** Every subagent role except `explore` has a self-contained file under `roles/`, dispatched by path plus parameter lines; orchestrator files never carry role instructions inline. Stage files, read in the orchestrator's context, assume `interaction.md` and the config are loaded.
- **Progress checklist for long runs.** Multi-phase files open with a copyable `- [ ]` checklist; phase names match the section headings.
- **Sections in execution order; define before use.** A contract governing later rules is an explicit forward reference ("wherever a rule below says to ask, report instead").
- **Match the construct to the control flow.** Routers use conditional bullets (`` `true` → … ``), pipelines numbered steps. One arrow separates a condition from its action.
- **Stopping rules, not continuation prose.** State when to stop ("Stop only at X, Y, Z"); continuing is then the default.
- **Templates as fenced blocks; examples over descriptions.** Output formats, dispatch prompts, and exact phrasings appear verbatim (`mise: approve goals`, "Bug fix or new feature?").
- **Progressive disclosure.** SKILL.md stays an overview under the 500-line ceiling; `references/` and `stages/` carry the depth, linked one level deep. `roles/` sits outside that count — role files are dispatched by path, not loaded as references.
- **"Do not" lists are the only sanctioned redundancy in a file.** They restate safety- and gate-critical rules; don't let convenience rules creep in.

## Agentic-loop patterns

- **Scripts over prose for fragile operations.** Anything with exact invariants — state transitions, hash verification — is code the model runs (`scripts/state.ts`), not rules applied by hand.
- **Pre-approve the skill's own machinery with `allowed-tools`.** An unattended run must not stall on a permission prompt for its state engine or checkpoint commits; grant the machinery only — cleanup deletion stays behind a prompt.
- **Fresh context per unit of work, self-contained instructions to match.** Each task, critique, review, and acceptance runs in a fresh subagent, catching what the author's context rationalizes away. Redundancy _across_ files is therefore intentional — task and role files repeat background instead of referencing siblings.
- **Tell every reviewer what counts as a finding.** A reviewer asked for gaps reports some even when work is sound, so role files name what counts (correctness, task spec, guides, checklist) and nothing else.
- **Small diffs, recorded rules.** 84% of diffs over 1,000 lines draw review findings, against 31% under 50 — hence the five-source-file cap per task. Codifying accepted review comments as rules, plus a 15-item pre-submission self-review checklist, eliminated recurrences across nine error classes — hence the checklist every implementer, fixer, and reviewer answers against its diff.
- **Orchestrators hold summaries, not sources.** An orchestrator reads the overview and subagent reports, never task files or source, so its context survives arbitrarily long plans.
- **Durable state outside the context window.** Anything a resumed session needs (approvals, completed tasks, decisions) lives in committed files; checkpoint commits make any checkout resumable.
- **Bounded retries with honest failure.** Loops carry an exit ("3 attempts with no new hypothesis → report failure").

## Process

- **Start from observed failures.** Before adding a skill or a section, run the task without it and note where the model goes wrong; keep a scenario as an informal eval.
- **Review by tracing, not by reading.** Walk the file as its executor would, with a concrete state in mind: it catches terms used before definition and one situation handled two ways.
- **Validate edits against real runs.** Exercise the skill after nontrivial changes: `/mise:next` on a throwaway branch, or a dry-run walkthrough of the changed file.
- **Test with every model that will run it.** Users run this on whatever model their session uses, so check both directions: enough guidance for the cheapest tier, no over-explaining for the strongest.

## Sources

- Skills: [authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) · [equipping agents](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- Claude Code: [best practices](https://code.claude.com/docs/en/best-practices) · [memory](https://code.claude.com/docs/en/memory) · [subagents](https://code.claude.com/docs/en/sub-agents)
- Anthropic: [long-running harnesses](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) · [context engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) · [behavioral rules](https://arxiv.org/abs/2607.13091) · [code review](https://claude.com/blog/code-review)
