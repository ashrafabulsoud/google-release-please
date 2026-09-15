# release-please CLI reference

Source: `docs/cli.md`, `docs/troubleshooting.md` (release-please 17.x).

```bash
npm i -g release-please
export GITHUB_TOKEN=ghp_...   # repo write scope
```

## Global flags (all commands)

| Flag | Notes |
|---|---|
| `--token` | required |
| `--repo-url=<owner>/<repo>` | required |
| `--target-branch` | branch to release from; default = repo default branch |
| `--dry-run` | report without opening PRs or tagging |
| `--debug` / `--trace` | verbose logging |
| `--api-url` / `--graphql-url` | GitHub Enterprise |

## Commands

### `bootstrap`: generate config + manifest via PR

```bash
release-please bootstrap --token=$GITHUB_TOKEN --repo-url=o/r \
  --release-type=node --initial-version=1.4.2 [--path=. --component=... --package-name=...]
```
Opens a PR adding `release-please-config.json` and `.release-please-manifest.json`. Accepts most per-package config keys as flags (`--bump-minor-pre-major`, `--changelog-path`, `--extra-files`, `--pull-request-title-pattern`, `--label`, `--release-label`, `--draft`, `--prerelease`, `--force-tag-creation`, `--versioning-strategy`, `--prerelease-type`, `--component-no-space`, `--version-file`, `--changelog-type`, `--changelog-sections`, `--changelog-host`, `--include-commit-authors`, `--draft-pull-request`, `--config-file`, `--manifest-file`).

Manual bootstrap alternative: commit a config with `"packages": {".": {}}` and a manifest of `{".": "<current version>"}`, optionally `bootstrap-sha` to limit the first changelog.

### `release-pr`: create or update the release PR

```bash
release-please release-pr --token=$GITHUB_TOKEN --repo-url=o/r [--dry-run --debug]
```
With a manifest config present, options come from the file. Extra flags: `--config-file`, `--manifest-file`, `--path` (release one component only), `--release-as`, `--draft-pull-request`, `--fork`, `--skip-labeling`.

Without a manifest you must pass `--release-type` and may pass `--package-name`, `--component`, `--monorepo-tags`, `--include-v-in-tags`, `--signoff`, `--extra-files`, `--version-file`, plus the same customization flags as `bootstrap`.

### `github-release`: tag + create GitHub Release for merged release PRs

```bash
release-please github-release --token=$GITHUB_TOKEN --repo-url=o/r [--draft --prerelease --force-tag-creation]
```
Finds merged PRs labelled `autorelease: pending`, tags the merge SHA, creates the release, swaps the label to `autorelease: tagged`. Not transactional; safe to re-run to finish partial failures. Run it before the next release PR merges.

### Deprecated: `manifest-pr`, `manifest-release`

Same behaviour as `release-pr` / `github-release` with a manifest. Will be removed in the next major.

## Debugging recipe

```bash
release-please release-pr --token=$GITHUB_TOKEN --repo-url=o/r \
  --target-branch=my-test-branch --debug --dry-run
```
Test config changes on a branch of the **real** repo. Forks do not carry PRs, releases, or tags, so release-please cannot find prior releases there.
