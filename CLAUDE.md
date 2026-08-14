# mise-claude-plugin

Never open a pull request for, or merge into another branch, any branch whose tree contains `.mise/` — that work is still in flight; run `/mise:next` on that branch to finish acceptance and cleanup first.

A user-visible change updates `README.md` in the same run — a new config section, a new stage, or changed stage behavior — using the exact terms the skill files use. The README is the user's only description of the config surface and the stage list, so a skill-file-only change ships an undocumented feature.

A user-visible change also bumps `version` in `.claude-plugin/plugin.json` in the same run — Claude Code installs and updates the plugin by version, so a change that ships without a bump never reaches an installed user, and no test or quality gate catches it.
