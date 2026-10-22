# Shared GitHub workflows

## Release workflow

`.github/workflows/release.yml` validates `Bump X.Y.Z` pull requests and publishes
GitHub Releases when those PRs merge. It operates on the **calling repository**,
using its PR event and `GITHUB_TOKEN`; no personal access token is required.

Add this caller to `.github/workflows/release.yml` in each repository:

```yaml
name: Release

on:
  pull_request:
    branches: [jazzy]
    types: [opened, synchronize, edited, reopened, ready_for_review, closed]

permissions:
  contents: read

jobs:
  release:
    permissions:
      contents: write
    uses: ROBOTIS-GIT/github-workflows/.github/workflows/release.yml@main
    with:
      source_branch: main
      target_branch: jazzy
```

The branch inputs default to `main` and `jazzy`. When changing `target_branch`,
also update the caller's `pull_request.branches` filter. Keep all event types so
changes to PR titles, descriptions, and commits trigger validation.

Every same-repository PR from `source_branch` to `target_branch` is validated,
regardless of its title. Validation requires a strict `Bump X.Y.Z` title, a version greater than the latest
published release, no existing tag or release for that version, and a description
containing more than whitespace or HTML comments. A missing or malformed title
fails validation instead of skipping it. PRs from other branch pairs or forks pass without creating
a release. Versions are stable three-part numbers; prerelease suffixes are not
supported. The latest release tag may have a `v` prefix, but new tags use `X.Y.Z`.

On merge, validation runs again before publishing a release at the PR's merge
commit, with the PR description as its release notes. Existing releases are not
overwritten; rerunning after publication fails duplicate/version validation.
The validation job has read-only permissions; only publication has write access.
This publishes GitHub Releases, not ROS packages.

## Rollout and updates

1. Publish this reusable workflow to this repository's `main` branch first.
2. Ensure repository/organization Actions settings permit callers to use it. If
   this repository is private, configure its Actions access policy and confirm
   the caller is eligible to access private reusable workflows.
3. Add the caller workflow to each consuming repository and its target branch.
4. After its first run, select the reusable workflow's validation check in the
   target branch's required status checks. Its displayed name may include the
   caller job prefix; replace any previous standalone validation check selection.

Callers reference `@main`, so updates merged here apply to their **subsequent
workflow runs** without editing every repository. Updating this repository does
not itself trigger releases in callers. Review and test central changes before
merging because all callers share this reference. Pinning a commit SHA is an
alternative when a repository needs fixed behavior instead of automatic updates.
