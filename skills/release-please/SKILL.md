---
name: release-please
description: Use when setting up, configuring, debugging, or operating release-please (googleapis/release-please CLI, GitHub Action, or bot) - release PRs, CHANGELOG generation, version bumps from conventional commits, release-please-config.json / .release-please-manifest.json, monorepo manifests, "Expected 1 releases, only found 0", missing release PR, autorelease labels, extra-files version annotations.
---

# release-please

## Overview

release-please turns Conventional Commits on a branch into a **Release PR** (version bump + CHANGELOG). Merging that PR makes it tag the commit and create a GitHub Release. It never publishes to package registries; gate a publish step on its outputs.

Core loop: `feat:` -> minor, `fix:` -> patch, `!`/`BREAKING CHANGE:` -> major. Only `feat`, `fix`, `deps` (and per-language extras like `docs` for Java/Python) are "releasable units"; `chore`/`build`/`ci` alone never open a PR.

Docs source: https://github.com/googleapis/release-please (`docs/`) and https://github.com/googleapis/release-please-action. Prefer these files, they are extracted from those docs (v17.x, action v4).

## Quick decisions

| Situation | Do this |
|---|---|
| New repo, any language | Manifest config (two JSON files) + GitHub Action `@v4`. See [reference/github-action.md](reference/github-action.md) |
| Need to test config changes | CLI with `--dry-run --debug --target-branch=<test-branch>` on the real repo, never a fork. See [reference/cli.md](reference/cli.md) |
| Bump versions in arbitrary files (docker-compose, VERSION, README) | `extra-files` + `x-release-please-version` annotations or typed json/yaml/toml/xml updaters. See [reference/config.md](reference/config.md#extra-files) |
| Monorepo, several packages | `packages` map with `component` per path; add `node-workspace` / `cargo-workspace` / `linked-versions` plugin as needed |
| Force a specific next version | Empty commit with `Release-As: x.y.z` in the body (preferred over `release-as` config, which is deprecated and sticky) |
| Fix release notes of a merged PR | Edit the PR body with a `BEGIN_COMMIT_OVERRIDE` ... `END_COMMIT_OVERRIDE` block (squash-merge repos only) |
| No release PR appears / tag not found | Follow [reference/troubleshooting.md](reference/troubleshooting.md) checklist in order |

## Minimal manifest setup

`release-please-config.json`:
```json
{
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json",
  "release-type": "node",
  "include-component-in-tag": false,
  "packages": { ".": {} }
}
```
`.release-please-manifest.json` (current released version, the only time you hand-edit it):
```json
{ ".": "1.4.2" }
```
Both files must exist at the tip of the target branch. `include-component-in-tag: false` gives `v1.4.2` tags for single-package repos; the default `true` gives `<component>-v1.4.2`.

## Common mistakes

- **Using `GITHUB_TOKEN` and expecting CI to run on the release PR or a `release.created` workflow to fire.** Events from `GITHUB_TOKEN` never trigger other workflows. Use a PAT or GitHub App token.
- **Tags are `v1.2.3` but release-please searches `<component>-v1.2.3`.** Set `include-component-in-tag: false`. Symptom: `Expected 1 releases, only found 0`.
- **Merge commits instead of squash.** Commit overrides and clean changelogs both assume squash-merge. Recommend squash.
- **Leaving `release-as` / `bootstrap-sha` / `last-release-sha` in config after they did their job.** `release-as` and `last-release-sha` are never ignored; remove them after the release lands.
- **Stale `autorelease: pending` label on an old PR** blocks new release PRs. Remove it and re-run.
- **Draft releases without `force-tag-creation: true`.** GitHub creates no tag for drafts, so the next run cannot find the previous release and dumps the whole history into the changelog.
- **`path` pointing at a file or using `.`/`..` segments.** Paths are directories relative to repo root.
- **Editing `.release-please-manifest.json` by hand after bootstrap.** Only release-please should write it after the first release.

## Reference files

- [reference/config.md](reference/config.md): every config key, per-package overrides, extra-files updaters, plugins, versioning strategies, changelog sections
- [reference/cli.md](reference/cli.md): `bootstrap`, `release-pr`, `github-release` flags
- [reference/github-action.md](reference/github-action.md): workflow YAML, inputs, outputs, publish gating, v3 to v4 migration
- [reference/troubleshooting.md](reference/troubleshooting.md): ordered checklists for the failure modes above
