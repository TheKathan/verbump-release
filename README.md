<p align="center">
  <a href="https://github.com/TheKathan/verbump-release/releases"><img alt="latest release" src="https://img.shields.io/github/v/release/TheKathan/verbump-release"></a>
  <img alt="license" src="https://img.shields.io/github/license/TheKathan/verbump-release">
  <img alt="runners: Linux, macOS" src="https://img.shields.io/badge/runners-Linux%20%7C%20macOS-555">
</p>

# verbump release

Releases the packages a merge changed, with [verbump](https://github.com/TheKathan/verbump). Each changed package gets its new version in its files and its changelog, then the action commits, tags, pushes, and publishes a GitHub release per package. Packages that didn't change stay where they are.

```yaml
on:
  push:
    branches: [main]

jobs:
  release:
    # the bot's own version commit must not release again
    if: "!startsWith(github.event.head_commit.message, 'chore(release):')"
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: read
    steps:
      - uses: actions/checkout@v7
        with:
          fetch-depth: 0
      - uses: TheKathan/verbump-release@v1
```

The repo needs a `verbump.toml` that lists its packages; [verbump's README](https://github.com/TheKathan/verbump#configuration) shows the format and the version files it understands on every stack.

## Where the bump comes from

The first of these that is set:

| Inputs | Bump |
|---|---|
| `bump` | `major`, `minor` or `patch` for every changed package |
| `previous-version` and `version` | the step between them, e.g. trunk-semver's two outputs |
| neither | the merged PR's branch: `feature/*` or `feat/*` minor, anything else patch, `release/<package>/X.Y.Z` exactly that version |

To let [trunk-semver](https://github.com/TheKathan/trunk-semver) pick it, run trunk-semver in a step before this one and pass `previous-version` and `version`. trunk-semver also pushes a repo-wide `vX.Y.Z` tag, so run it against a local mirror of the repo: see [verbump's own release workflow](https://github.com/TheKathan/verbump/blob/main/.github/workflows/release.yml) for the three steps.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `bump` | `""` | `major`, `minor` or `patch` for every changed package. |
| `previous-version` | `""` | With `version`: the bump is the step between them. |
| `version` | `""` | See `previous-version`. |
| `source-branch` | the merged PR's branch | Picks the bump when no bump input is set; `release/…` gives an exact version. |
| `config` | `verbump.toml` | Relative to `working-directory`. |
| `working-directory` | `.` | The repo root. |
| `push` | `"true"` | `"false"` writes the files and the plan only (a dry run). |
| `github-release` | `"true"` | Publish a GitHub release per package, with its changelog section as the body. |
| `push-branch` | the pushed branch | Where the release commit goes. |
| `github-token` | `github.token` | For the PR lookup and the GitHub releases. |
| `verbump-version` | `1` | verbump to install with [setup-verbump](https://github.com/TheKathan/setup-verbump): `1`, an exact version, or a pip requirement. |
| `install` | `"true"` | `"false"` uses a `verbump` already on the PATH. |

## Outputs

| Output | Meaning |
|---|---|
| `released` | `true` when at least one package was released (on a dry run, would have been). |
| `tags` | JSON array of the new tags, e.g. `["api/v1.5.0"]`. Test it with `contains(steps.x.outputs.tags, '"api/v')`. |
| `plan` | Path to the plan JSON: `bump`, `commit_message` and `releases` with `name`, `path`, `previous`, `version`, `tag`, `major_tag`, `changelog`, `reason`, `changed`. |

## Notes

- **Protected branches:** if a ruleset requires PRs, check out with a write deploy key (`ssh-key: ${{ secrets.RELEASE_DEPLOY_KEY }}`) that only it may bypass, and keep the key in an environment limited to the release branch.
- **Tags:** an existing exact tag stops the run; it is never moved. With `major_tag = true` in `verbump.toml`, a `vX` tag follows the latest release.
- **Preview on PRs:** [verbump-check](https://github.com/TheKathan/verbump-check) shows what merging a PR will release, and fails early when the release would fail.

Runs on Linux and macOS runners. MIT licensed.
