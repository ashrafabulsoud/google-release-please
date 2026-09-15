# release-please configuration reference

Source: `docs/manifest-releaser.md`, `docs/customizing.md`, `schemas/config.json` (release-please 17.x). Schema URL for editor validation:
`https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json`

Two files, both at the tip of the target branch:

- `release-please-config.json`: configuration. Must define at least one entry under `packages`.
- `.release-please-manifest.json`: map of package path -> last released version. Start with `{}` or seed versions once during bootstrap; release-please writes it afterwards.

## Top-level-only keys

| Key | Default | Purpose |
|---|---|---|
| `packages` | required | Map of `<path>` -> per-package config. `"."` is the repo root. Paths are directories relative to repo root, no `.`/`..` segments. |
| `bootstrap-sha` | none | Full SHA. First run collects commits back to (exclusive) this SHA. Ignored once any release PR has merged; remove afterwards. |
| `last-release-sha` | none | Full SHA to treat as the previous release marker. Use after a bad release PR was merged. **Never ignored**: remove once a good release merges. |
| `plugins` | `[]` | See Plugins below. |
| `separate-pull-requests` | `false` | One PR per package instead of one combined manifest PR. |
| `group-pull-request-title-pattern` | `chore: release ${branch}` | Title for the combined PR. |
| `always-update` | `false` | Update the release PR on every run, not only when notes change. |
| `always-link-local` | `true` | `node-workspace`: link local deps even across breaking SemVer ranges. |
| `release-search-depth` | `400` | How many recent releases to scan for the previous release SHA. |
| `commit-search-depth` | `500` | How many commits to scan back. |
| `commit-batch-size` | `10` | GraphQL page size when fetching commits. |
| `sequential-calls` | `false` | Serialize GitHub API calls (throttling relief for many packages). |
| `skip-labeling` | `false` | Do not apply labels to PRs. |
| `signoff` | none | `"Name <email>"` to add `Signed-off-by`. |
| `label` | `autorelease: pending` | Label(s) on open release PRs (comma-separated). |
| `release-label` | `autorelease: tagged` | Label(s) after tagging. |

## Per-package keys (also valid at top level as defaults)

| Key | Default | Purpose |
|---|---|---|
| `release-type` | `node` | Strategy. See list below. |
| `component` | derived from package name | Name used in branch names and tags (`<component>-v1.2.3`). Set explicitly in monorepos. |
| `package-name` | strategy lookup | Required for strategies that cannot read the name from source (e.g. `python`, `simple`). |
| `changelog-path` | `CHANGELOG.md` | Relative to the package dir. |
| `changelog-type` | `default` | `default` (conventional-changelog) or `github` (GitHub release-notes API). |
| `changelog-sections` | see below | Array of `{type, section, hidden?}`. |
| `changelog-host` | `https://github.com` | For GitHub Enterprise links. |
| `include-commit-authors` | `false` | Append `(@user)` to entries. |
| `skip-changelog` | `false` | Do not touch the changelog. |
| `versioning` | `default` | Versioning strategy. See list below. |
| `bump-minor-pre-major` | `false` | Breaking change bumps minor while `< 1.0.0`. |
| `bump-patch-for-minor-pre-major` | `false` | `feat` bumps patch while `< 1.0.0`. |
| `prerelease-type` | none | e.g. `beta`; used by the `prerelease` versioning strategy. |
| `prerelease` | `false` | Mark GitHub release as prerelease; also required for the `prerelease` versioning strategy to emit `-beta.N` versions. |
| `release-as` | none | **Deprecated and sticky.** Prefer a `Release-As: x.y.z` commit footer. `""` on a package resets to conventional bumping when a top-level default exists. |
| `initial-version` | `0.0.0` (node: `0.1.0`) | First version when the package has never been released. |
| `draft` | `false` | Create GitHub releases as drafts. Pair with `force-tag-creation`. |
| `force-tag-creation` | `false` | Create the git tag immediately even for drafts. |
| `skip-github-release` | `false` | Do not create GitHub releases (you must still tag somehow). |
| `draft-pull-request` | `false` | Open release PR as draft. |
| `extra-label` | none | Extra labels on newly opened PRs. |
| `include-component-in-tag` | `true` | `false` -> tags are `v1.2.3`. Single-package repos almost always want `false`. In manifest mode this config key is what counts; the Action input of the same name (default `false`) applies to the no-config `release-type` path. |
| `include-v-in-tag` | `true` | `false` -> tags are `1.2.3`. |
| `include-v-in-release-name` | `true` | GitHub release title with/without `v`. |
| `tag-separator` | `-` | Between component and version in tags. |
| `pull-request-title-pattern` | `chore${scope}: release${component} ${version}` | Placeholders: `${scope}` (target branch), `${component}`, `${version}`, `${branch?}`. |
| `component-no-space` | `false` | Drop the leading space before `${component}`. Changing on an existing PR can break parsing and open a duplicate PR. |
| `pull-request-header` | `:robot: I have created a release *beep* *boop*` | |
| `pull-request-footer` | `This PR was generated with Release Please. See documentation.` | |
| `extra-files` | `[]` | Additional files to bump. See below. |
| `exclude-paths` | `[]` | Skip commits whose files all fall under these paths. |
| `version-file` | strategy default | `ruby` (`version.rb`) and `simple` (`version.txt`) only. |
| `date-format` | none | strftime pattern for the generic updater. |
| `snapshot-label` / `skip-snapshot` | | `java`/`maven` strategies. |

### Default changelog sections

```json
[
  {"type": "feat", "section": "Features"},
  {"type": "fix", "section": "Bug Fixes"},
  {"type": "perf", "section": "Performance Improvements"},
  {"type": "revert", "section": "Reverts"},
  {"type": "chore", "section": "Miscellaneous Chores", "hidden": true},
  {"type": "docs", "section": "Documentation", "hidden": true},
  {"type": "style", "section": "Styles", "hidden": true},
  {"type": "refactor", "section": "Code Refactoring", "hidden": true},
  {"type": "test", "section": "Tests", "hidden": true},
  {"type": "build", "section": "Build System", "hidden": true},
  {"type": "ci", "section": "Continuous Integration", "hidden": true}
]
```
Hidden types still appear if the commit is breaking. Override the whole array to unhide `docs`/`chore` etc.

## Release types (strategies)

`bazel`, `dart`, `elixir`, `expo`, `go`, `helm`, `java`, `krm-blueprint`, `maven`, `node`, `ocaml`, `php`, `python`, `r`, `ruby`, `rust`, `sfdx`, `simple`, `terraform-module`.

What each bumps beyond `CHANGELOG.md`:

| Type | Files |
|---|---|
| `node` | `package.json` version (+ `package-lock.json`, `npm-shrinkwrap.json`) |
| `python` | `pyproject.toml`, `setup.py`, `setup.cfg`, `<pkg>/__init__.py`, `version.py` |
| `go` | changelog only (tags drive Go modules) |
| `rust` | `Cargo.toml` (crate or workspace; workspaces need `cargo-workspace` plugin) + `Cargo.lock` |
| `simple` | `version.txt` (or `version-file`) |
| `helm` | `Chart.yaml` |
| `php` | `composer.json` |
| `ruby` | `version.rb` via `version-file` |
| `dart` | `pubspec.yaml` |
| `elixir` | `mix.exs` |
| `expo` | `package.json`, `app.json` |
| `java` / `maven` | `maven` updates all `pom.xml`; `java` needs `extra-files`. Both open a follow-up SNAPSHOT bump PR labelled `autorelease: snapshot`. |
| `terraform-module` | version in `README.md` |
| `sfdx` | `sfdx-project.json` |

`go`, `java`, `python` treat `docs:` as releasable in addition to `feat`/`fix`/`deps`.

## Versioning strategies (`versioning`)

| Value | Behaviour |
|---|---|
| `default` | breaking -> major, feat -> minor, fix -> patch |
| `always-bump-patch` | patch only (backport/LTS branches) |
| `always-bump-minor` | minor only |
| `always-bump-major` | major only |
| `prerelease` | `1.2.0-beta01` -> `1.2.0-beta02`, or with `prerelease-type` set `1.2.1` -> `1.3.0-beta`. Requires `"prerelease": true`. |
| `service-pack` | Maven `1.2.3-sp.1` style (Java backports) |

## Extra files

`extra-files` accepts strings or objects with an explicit `type` (verified in `src/strategies/base.ts`):

- **Bare string** ending in `.json` / `.yaml` / `.yml` / `.toml` / `.xml`: runs the typed updater on `$.version` (XML: `/*/version`) **and** the generic annotation updater on the same file. So `"docker-compose.yml"` as a string works with `# x-release-please-version` comments; the missing top-level `version` key is simply ignored.
- **Bare string** with any other extension: generic annotation updater only.
- **Object** `{ "type": ..., "path": ... }`: exactly that updater. Add `"glob": true` to expand `path` as a glob. A path starting with `/` is relative to the repo root instead of the package dir.

Generic markers also include `x-release-please-date` and `x-release-please-version-date`, formatted with `date-format` (strftime).

### Generic annotations (any text file)

```json
{ "extra-files": ["VERSION", "docker-compose.yml", "README.md"] }
```
Mark the line to rewrite with a comment containing one of:

- `x-release-please-version` (full `1.2.3`)
- `x-release-please-major` / `-minor` / `-patch`

Block form: a line containing `x-release-please-start-version` (or `-major`/`-minor`/`-patch`) ... `x-release-please-end`. Everything version-shaped inside the block is replaced.

Example `docker-compose.yml`:
```yaml
services:
  api:
    image: ghcr.io/acme/api:1.2.3 # x-release-please-version
```
The generic updater only rewrites annotated lines, so a bare `VERSION` file with just `1.2.3` in it cannot be handled by `extra-files`. Options: put the marker on the line (`1.2.3 # x-release-please-version`) and have readers strip it, use the block form with the markers on their own lines, or make the package `release-type: simple` with `"version-file": "VERSION"` (the `simple` strategy owns that file and needs no annotation, but it replaces the language strategy, so a Node package would lose its `package.json` bump; in that case stick to the annotation options).

Force the generic updater for a file whose extension would otherwise pick a typed updater:
```json
{ "extra-files": [{ "type": "generic", "path": "chart/values.yaml" }] }
```

### Typed updaters (no annotations needed)

| `type` | Target selector | Default when only a path is given |
|---|---|---|
| `json` | `"jsonpath": "$.a.b.version"` | `.json` -> `$.version` |
| `yaml` | `"jsonpath": "$.services.api.version"` | `.yaml`/`.yml` -> `$.version` |
| `toml` | `"jsonpath": "$.package.version"` | `.toml` -> `version` |
| `xml` | `"xpath": "//project/version"` | `.xml` -> `version` element |
| `pom` | none | `/project/version` or parent version |

```json
{
  "extra-files": [
    { "type": "json", "path": "app/manifest.json", "jsonpath": "$.build.version" },
    { "type": "yaml", "path": "chart/Chart.yaml", "jsonpath": "$.appVersion" },
    { "type": "toml", "path": "pyproject.toml", "jsonpath": "$.tool.poetry.version" },
    { "type": "xml", "path": "pkg.nuspec", "xpath": "//package/metadata/version" }
  ]
}
```
Typed updaters replace the whole field value, so `image: ghcr.io/x:1.2.3` must use the generic annotation, not `yaml`.

Paths are relative to the **package** directory, not the repo root.

## Plugins

```json
"plugins": ["node-workspace", { "type": "linked-versions", "groupName": "core", "components": ["a", "b"] }]
```

| Plugin | Purpose | Options |
|---|---|---|
| `node-workspace` | Bump local npm workspace dependents; patch-bumps packages whose deps changed | `updatePeerDependencies: true`, top-level `always-link-local` |
| `cargo-workspace` | Same for Cargo workspaces; updates `Cargo.lock`. Required for Rust monorepos in manifest mode | `merge: false` when combined with `linked-versions` |
| `maven-workspace` | Same for multi-module Maven | `considerAllArtifacts: false` |
| `linked-versions` | Keep a group of components on the same version (highest wins) | `groupName`, `components` |
| `sentence-case` | Capitalize first word of changelog entries (keeps gRPC etc.) | |
| `group-priority` | Only open PRs for the highest-priority group present (e.g. Java `snapshot`) | `groups: ["snapshot"]` |

Plugins run after per-package strategies and before the PR is created; order matters.

## Monorepo example

```json
{
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json",
  "release-type": "node",
  "separate-pull-requests": false,
  "plugins": ["node-workspace"],
  "packages": {
    "packages/core": { "component": "core" },
    "packages/cli": { "component": "cli", "extra-files": ["README.md"] },
    "services/api": { "release-type": "python", "package-name": "acme-api", "changelog-path": "docs/CHANGES.md" }
  }
}
```
`.release-please-manifest.json`:
```json
{ "packages/core": "2.1.0", "packages/cli": "0.9.3", "services/api": "1.0.0" }
```
Tags become `core-v2.1.1`, `cli-v0.9.4`, `acme-api-v1.0.1`. Commits count toward a package only if they touch files under its path; `"."` receives every commit.

## Commit-message controls

- `Release-As: 2.0.0` in a commit body (case-insensitive) forces that version: `git commit --allow-empty -m "chore: release 2.0.0" -m "Release-As: 2.0.0"`.
- Multiple changes in one squash commit: add extra `type(scope): message` lines plus optional `BREAKING-CHANGE:` footers at the **bottom** of the body.
- `BEGIN_COMMIT_OVERRIDE` / `END_COMMIT_OVERRIDE` in a merged PR body replaces that commit's message on the next run (squash-merge only).
