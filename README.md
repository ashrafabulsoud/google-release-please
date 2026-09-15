# google-release-please

An agent skill for [release-please](https://github.com/googleapis/release-please), Google's tool that turns Conventional Commits into release PRs, CHANGELOG entries, version bumps, tags, and GitHub Releases.

The skill gives coding agents (Claude Code, Cursor, Codex, and any other [Agent Skills](https://agentskills.io) compatible tool) an accurate, offline reference for configuring and debugging release-please, so they stop guessing config keys or inventing flags.

## Install

```bash
npx skills add https://github.com/ashrafabulsoud/google-release-please
```

Useful variants:

```bash
# install for every detected agent without prompts
npx skills add https://github.com/ashrafabulsoud/google-release-please --all

# user-level install instead of the current project
npx skills add https://github.com/ashrafabulsoud/google-release-please -g

# see what the repo exposes first
npx skills add https://github.com/ashrafabulsoud/google-release-please --list
```

The repository is private, so the `skills` CLI needs a git credential that can read it. A GitHub CLI login (`gh auth login`) or an SSH key configured for GitHub is enough.

Manual install for Claude Code only:

```bash
git clone git@github.com:ashrafabulsoud/google-release-please.git
ln -s "$PWD/google-release-please/skills/release-please" ~/.claude/skills/release-please
```

## What the skill covers

| Area | Where | Highlights |
|---|---|---|
| Core concepts and quick decisions | `SKILL.md` | Release PR lifecycle, releasable commit types, minimal two-file manifest setup, common mistakes |
| Configuration | `reference/config.md` | Every key in `release-please-config.json`, per-package overrides, all release types and what files they bump, versioning strategies, default changelog sections, `extra-files` updaters and annotations, plugins, monorepo example |
| CLI | `reference/cli.md` | `bootstrap`, `release-pr`, `github-release`, global flags, dry-run debugging recipe |
| GitHub Action | `reference/github-action.md` | v4 workflow YAML, inputs, outputs, publish gating for root and monorepo paths, the `GITHUB_TOKEN` trap, v3 to v4 migration table |
| Troubleshooting | `reference/troubleshooting.md` | Ordered checklists: no release PR, `Expected 1 releases, only found 0`, whole history in changelog, wrong version, wrong notes, merged but untagged, duplicate PRs |

## When it triggers

The skill description matches requests such as:

- "Set up release-please for this monorepo"
- "release-please is not opening a release PR"
- "Make release-please bump the version in docker-compose.yml"
- "Why are tags `my-app-v1.2.3` instead of `v1.2.3`?"
- "Gate npm publish on the release-please action output"
- "Migrate our release-please-action workflow from v3 to v4"

Agents that support slash commands can also invoke it directly as `/release-please`.

## Example

Prompt:

> release-please fails with `looking for tagName: my-app-v1.2.3 / Expected 1 releases, only found 0`. Our tags are `v1.2.3`.

With the skill loaded, the agent answers from `reference/troubleshooting.md`:

```json
{ "include-component-in-tag": false }
```

and explains that the default tag pattern is `<component>-v<version>`, that `include-v-in-tag: false` is the companion for tags without a `v`, and that the key belongs in the manifest config rather than the Action input when a manifest is in use.

## Repository layout

```
skills/
  release-please/
    SKILL.md                     entry point, loaded first
    reference/
      config.md
      cli.md
      github-action.md
      troubleshooting.md
LICENSE
README.md
```

Skills are discovered from `skills/*/SKILL.md`, the layout the `skills` CLI expects.

## Sources and versions

Content was extracted from these upstream sources and verified against the source code where the docs were ambiguous:

| Source | Version |
|---|---|
| `googleapis/release-please` docs (`README.md`, `docs/manifest-releaser.md`, `docs/customizing.md`, `docs/cli.md`, `docs/troubleshooting.md`, `docs/design.md`), `schemas/config.json`, `src/strategies/base.ts`, `src/util/filter-commits.ts` | 17.11.2 |
| `googleapis/release-please-action` `README.md`, `action.yml` | v4 |

## Contributing

1. Edit the files under `skills/release-please/`.
2. Keep `SKILL.md` short; put detail in `reference/`.
3. Update the description in the `SKILL.md` frontmatter only with triggering conditions, never with a summary of the workflow.
4. Verify with a fresh agent session: ask it a question the change is meant to answer and confirm it cites the right file.

## License

This repository is MIT licensed. The documentation it summarizes comes from release-please, which is Apache-2.0.
