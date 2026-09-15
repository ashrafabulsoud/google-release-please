# googleapis/release-please-action (v4) reference

Source: release-please-action `README.md`, `action.yml`. Runs on `node24`.

## Workflow

```yaml
name: release-please
on:
  push:
    branches: [main]

permissions:
  contents: write
  issues: write
  pull-requests: write

jobs:
  release-please:
    runs-on: ubuntu-latest
    steps:
      - uses: googleapis/release-please-action@v4
        id: release
        with:
          token: ${{ secrets.RELEASE_PLEASE_TOKEN }}   # PAT or GitHub App token, see below
          # Either a manifest config (recommended, default when release-type is unset):
          config-file: release-please-config.json
          manifest-file: .release-please-manifest.json
          # ...or the zero-config path:
          # release-type: node
```

Also enable repo Settings > Actions > General > "Allow GitHub Actions to create and approve pull requests".

### Token

`token` defaults to `GITHUB_TOKEN`. Anything created with `GITHUB_TOKEN` (the release PR, the tag, the release) **does not trigger other workflows**: CI will not run on the release PR and `on: release` / `on: push: tags` workflows will not fire. Use a Personal Access Token or a GitHub App installation token stored as a secret when you need those. `fork: true` also requires a non-default token.

## Inputs

| Input | Default | Notes |
|---|---|---|
| `token` | `github.token` | |
| `release-type` | unset | Set only for the no-config path; leaving it unset selects manifest mode |
| `config-file` | `release-please-config.json` | |
| `manifest-file` | `.release-please-manifest.json` | |
| `target-branch` | detected | Use `${{ github.ref_name }}` to serve several release branches from one workflow |
| `path` | root | Release from a subdirectory (no-config path) |
| `repo-url` | current repo | |
| `github-api-url`, `github-graphql-url` | github.com | GHES |
| `fork` | `false` | PR from a fork |
| `include-component-in-tag` | `false` | No-config path only. With a manifest config, set the key in `release-please-config.json` (library default there is `true`) |
| `skip-github-release` | `false` | Only open PRs (replaces v3 `command: release-pr`) |
| `skip-github-pull-request` | `false` | Only tag releases (replaces v3 `command: github-release`) |
| `skip-labeling` | `false` | |
| `changelog-host` | `github.server_url` | |
| `versioning-strategy` | `default` | |
| `release-as` | unset | |
| `proxy-server` | unset | `host:port` |

Everything else (`draft`, `prerelease`, `force-tag-creation`, `extra-files`, labels, title patterns, plugins, ...) has no Action input in v4 and must go in `release-please-config.json`.

## Outputs

Always:

| Output | Meaning |
|---|---|
| `releases_created` | `true` if any release was created |
| `paths_released` | JSON array of paths released (`[]` if none) |
| `prs_created` | `true` if any PR was created or updated |
| `pr` / `prs` | JSON PullRequest object / array (unset when nothing created) |

Root component (`.`) adds: `release_created`, `tag_name`, `version`, `major`, `minor`, `patch`, `sha`, `html_url`, `upload_url`, `body`.

Non-root paths prefix each of those with `<path>--`, e.g. `packages/cli--release_created`, `packages/cli--tag_name`. Paths containing `/` need bracket access:

```yaml
if: ${{ steps.release.outputs['packages/cli--release_created'] }}
```

## Gating a publish step

```yaml
      - uses: actions/checkout@v4
        if: ${{ steps.release.outputs.release_created }}
      - uses: actions/setup-node@v4
        if: ${{ steps.release.outputs.release_created }}
        with:
          node-version: 20
          registry-url: https://registry.npmjs.org
      - run: npm ci && npm publish
        if: ${{ steps.release.outputs.release_created }}
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```
For a monorepo, gate on `releases_created` for "anything released" or on `<path>--release_created` per package, and iterate `fromJSON(steps.release.outputs.paths_released)` in a matrix if you need one job per package.

Attach assets: `gh release upload ${{ steps.release.outputs.tag_name }} ./dist/*.zip` with `GITHUB_TOKEN` in env.

Floating major/minor tags (for publishing GitHub Actions): after `release_created`, delete and re-push `v${major}` and `v${major}.${minor}` tags using `steps.release.outputs.major` / `.minor`.

## Multiple release branches

Trigger on each branch and pass `target-branch: ${{ github.ref_name }}`. Each branch needs its own config/manifest at its tip. Backport branches usually set `versioning: always-bump-patch`.

## v3 to v4 migration

| v3 | v4 |
|---|---|
| `command: manifest` | remove `command`, leave `release-type` unset |
| `command: manifest-pr` | `skip-github-release: true`, no `release-type` |
| `command: release-pr` | `skip-github-release: true` |
| `command: github-release` | `skip-github-pull-request: true` |
| `default-branch` | `target-branch` |
| Package inputs (`changelog-path`, `component`, `package-name`, `bump-minor-pre-major`, `extra-files`, `pull-request-title-pattern`, ...) | move into `release-please-config.json` under `packages[path]` or top level |
| Root inputs (`bootstrap-sha`, `last-release-sha`, `plugins`, `always-update`, `separate-pull-requests`, `signoff`, `skip-labeling`, `sequential-calls`, ...) | top level of `release-please-config.json` |
