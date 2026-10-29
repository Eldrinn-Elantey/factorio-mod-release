# Factorio Mod Release

A reusable GitHub Actions workflow that releases a Factorio mod when you push a version tag. [Auto Research 2](https://github.com/Eldrinn-Elantey/auto-research-2), [Advanced Fluid Handling Tweaks](https://github.com/Eldrinn-Elantey/advanced-fluid-handling-tweaks) and [Ores Avoid Water](https://github.com/Eldrinn-Elantey/ores-avoid-water) use it.

## What a release does

Pushing a tag like `1.2.3` runs these steps in order:

1. Stops if the tag is not the tip of the default branch, since the changelog commit at the end goes there.
2. Replaces the `Version: 0.0.0` placeholder at the top of `changelog.txt` with the tag and today's date.
3. Runs the mod's own `release.ps1`, which builds the zip into `build/`.
4. Creates a GitHub release with the zip, using the changelog entry as release notes.
5. Uploads the zip to the mod portal, if `FACTORIO_API_KEY` is set.
6. Adds a new `0.0.0` placeholder to the changelog and commits it to the default branch.
7. Copies `README.md` to the mod portal page, if `FACTORIO_API_KEY` is set. The README becomes the description, the `info.json` description becomes the summary, and the repository becomes the source link. This runs last so a failed sync cannot block the changelog commit.

With `readme-only: true` the workflow skips steps 1 to 6 and only copies the README from the default branch. Without the API key that run fails.

## What the mod repository needs

- `info.json` and `changelog.txt` in the root, with a `Version: 0.0.0` entry at the top of the changelog.
- `release.ps1` in the root. It reads the version from `GITHUB_REF_NAME`, stamps it into `info.json` and writes exactly one zip into `build/`. The script stays in the mod so each mod picks its own files.
- A mod page on the portal. You create it by hand with the first upload, since the API only adds releases to an existing mod and edits its details.
- A `FACTORIO_API_KEY` secret with the "ModPortal: Upload Mods" and "ModPortal: Edit Mods" scopes, if the release should reach the portal.

## Caller workflow

Put this in `.github/workflows/release.yml` of the mod:

```yaml
name: Release

on:
  push:
    tags:
      - '[0-9]+.[0-9]+.[0-9]+'
  # Run by hand to copy README.md to the mod portal page without a release.
  workflow_dispatch:

permissions:
  contents: write

jobs:
  release:
    uses: Eldrinn-Elantey/factorio-mod-release/.github/workflows/release.yml@v1
    with:
      readme-only: ${{ github.event_name == 'workflow_dispatch' }}
    secrets:
      FACTORIO_API_KEY: ${{ secrets.FACTORIO_API_KEY }}
```

Set `permissions` in the caller: a called workflow can only lower the permissions it receives.

## Versions

`v1` points at the latest compatible version, so a fix reaches all three mods without a PR in each. Each version also gets a fixed tag such as `v1.0.0`. Changes that break callers go to `v2`.
