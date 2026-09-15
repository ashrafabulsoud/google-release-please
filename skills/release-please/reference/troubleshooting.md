# release-please troubleshooting

Source: `README.md`, `docs/troubleshooting.md`, `docs/manifest-releaser.md`.

First step for any problem: reproduce locally with the CLI.
```bash
release-please release-pr --token=$GITHUB_TOKEN --repo-url=o/r --debug --dry-run [--target-branch=b]
```

## No release PR is opened

Check in this order:

1. **Releasable commits since last release?** Only `feat`, `fix`, `deps` count (plus `docs` for Go/Java/Python). `chore:`, `build:`, `ci:`, `refactor:` alone produce nothing. Non-conventional messages are ignored. Squash-merge PR titles must themselves be conventional.
2. **Commits touch the package path?** In a manifest, a commit counts for `packages/x` only if it changes files under `packages/x`. Check `exclude-paths`.
3. **Stale `autorelease: pending` or `autorelease: triggered` label** (`triggered` is set by the GitHub App while a release job runs) on an older PR. release-please assumes a release is still in flight and will not open another. Remove the label from the stale PR and re-run.
4. **Closed-then-reopened release PR.** Closing adds `autorelease: closed` and reopening does not restore `autorelease: pending`. Remove `autorelease: closed` and add `autorelease: pending`; GitHub App users also add `release-please:force-run` (that label is only read by the App, not the Action or CLI), Action users re-run the workflow.
5. **Config/manifest not at the tip of the target branch**, or `packages` empty, or path points at a file.
6. **Workflow permissions**: `contents: write`, `pull-requests: write`, `issues: write`, and the repo setting "Allow GitHub Actions to create and approve pull requests".
7. **Re-run**: Action -> re-run the failed workflow; GitHub App -> add label `release-please:force-run` to the merged PR; CLI -> run `release-pr` again.

## `looking for tagName: <component>-v1.2.3` / `Expected 1 releases, only found 0`

Tags in the repo are `v1.2.3` but release-please expects `<component>-v1.2.3`. Fix:
```json
{ "include-component-in-tag": false }
```
If tags have no `v` at all, also `"include-v-in-tag": false`. If the component name itself is wrong, set `component` explicitly on the package. If the tag exists but the release does not, release-please falls back to tags, but only within `commit-search-depth` (500) commits and `release-search-depth` (400) releases; raise them for large repos.

## Changelog contains the whole history / wrong previous version

- Manifest version wrong or missing: seed `.release-please-manifest.json` with the real current version, or set `bootstrap-sha` (full SHA of the commit **before** the first one you want included) for the first run.
- Draft releases without `force-tag-creation: true`: GitHub creates no tag for drafts, so the previous release cannot be located. Publish the drafts or enable the flag.
- A bad release PR was merged: set `last-release-sha` to the last good release commit, run once, then **remove it** (it is never auto-ignored).

## Version bump is wrong

- Version stuck: a `release-as` key left in config or a `Release-As:` footer being re-read. Remove the config key after the release lands.
- Want `feat` -> patch or breaking -> minor while `< 1.0.0`: `bump-patch-for-minor-pre-major` / `bump-minor-pre-major`.
- Force a version once: `git commit --allow-empty -m "chore: release 2.0.0" -m "Release-As: 2.0.0"`.
- Prerelease versions not generated: `prerelease` versioning strategy requires `"prerelease": true` in config.

## Release notes are wrong

- Squash-merge repos: edit the merged PR body and add
  ```
  BEGIN_COMMIT_OVERRIDE
  feat: correct description

  fix: another entry
  END_COMMIT_OVERRIDE
  ```
  Next run uses that as the commit message. Does not work with merge commits.
- Types missing from the changelog (`docs`, `chore`): they are `hidden: true` by default; override `changelog-sections`.
- Several changes in one squash: add extra `type: message` lines at the bottom of the commit body.

## Release PR merged but no tag/release

- Action with only `skip-github-release: true`, or CLI runs only `release-pr`: run `github-release` (or a second workflow / job without the skip).
- Run failed midway: `github-release` is safe to re-run; it resumes untagged PRs labelled `autorelease: pending`.
- Two release PRs merged before tagging: only the latest is found. Tag the earlier one manually.
- The merged PR carries `autorelease: closed` (it was closed and reopened before merging) so `github-release` never sees it: swap the label to `autorelease: pending` on the merged PR and re-run.

## Downstream workflows do not run

`GITHUB_TOKEN`-created PRs, tags and releases never trigger other workflows. Provide a PAT or GitHub App token via `token`. Alternatively run publish steps in the same job gated on `steps.release.outputs.release_created`.

## Duplicate release PRs

- Changed `pull-request-title-pattern` or `component-no-space` while a release PR was open; release-please could not parse the old one. Close the old PR and remove its label.
- Two components resolved to the same name; set distinct `component` values.

## Testing changes safely

Push the config change to a branch on the **same** repository and run with `--target-branch=<branch> --dry-run --debug`. Forks lack the PRs, releases, and tags release-please reads.
