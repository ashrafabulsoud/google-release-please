# google-release-please

A Claude Code skill for working with [release-please](https://github.com/googleapis/release-please): setting up manifest configs, the GitHub Action, the CLI, and debugging stuck release PRs.

Extracted from the release-please docs (v17.x), config schema, and source, plus the release-please-action v4 README.

## Install

Copy or symlink the skill into your personal skills directory:

```bash
git clone https://github.com/ashrafabulsoud/google-release-please.git
ln -s "$PWD/google-release-please/skills/release-please" ~/.claude/skills/release-please
```

Then use `/release-please` in Claude Code, or just ask about release-please; the skill triggers on its description.

## Layout

- `skills/release-please/SKILL.md`: overview, decision table, minimal setup, common mistakes
- `skills/release-please/reference/config.md`: every config key, strategies, extra-files updaters, plugins
- `skills/release-please/reference/cli.md`: `bootstrap`, `release-pr`, `github-release`
- `skills/release-please/reference/github-action.md`: workflow, inputs, outputs, v3 to v4 migration
- `skills/release-please/reference/troubleshooting.md`: ordered checklists for common failures

## License

Documentation derived from release-please, Apache-2.0. This repo is MIT.
