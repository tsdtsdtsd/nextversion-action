# nextversion-action

![Build Status](https://github.com/tsdtsdtsd/nextversion-action/actions/workflows/test.yml/badge.svg)

> Automatic semantic versioning action

nextversion-action uses [nextversion](https://github.com/tsdtsdtsd/nextversion) to detect current application version based on git tags and suggesta a bumped version based on [Semantic Versioning](https://semver.org/) and [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).


## Action Inputs (Configuration)

| input name | description | default |
| ---------- | ----------- | ------- |
| `repo` | Path to git repository | `.` |
| `default-current` | Default current version if none could be detected from tags | `v0.0.0` |
| `prestable-mode` | Disable major increments for versions prior to 1.0.0 (in-development/prestable phase; see [Prestable](#prestable) | `false` |

## Action Outputs

| input name | description | example |
| ---------- | ----------- | ------- |
| `hasCurrentVersion` | "false" if nextversion-action wasn't able to determine a current version | "true" |
| `hasNextVersion` | "false" if nextversion-action wasn't able to determine a next version, for example when there is no applicable conventional commit messages since current version | "true" |
| `currentVersion` | determined current version according to newest git tag | `v1.2.3` |
| `currentVersionStrict` | same as `currentVersion`, but without the `v` prefix | `1.2.3` |
| `nextVersion` | calculated next version | `v1.3.0` |
| `nextVersionStrict` | same as `nextVersion`, but without the `v` prefix | `1.3.0` |
| `prereleaseVersion` | calculated prerelease version | `v1.3.0-rc+dev.4fa53ce` |
| `prereleaseVersionStrict` | same as `prereleaseVersion`, but without the `v` prefix | `1.3.0-rc+dev.4fa53ce` |

## Usage Example

```yaml
name: Example workflow

on: push

jobs:
  example:
    name: Example
    runs-on: ubuntu-latest

    steps:

    # We need to clone the repository.
    # It's very important that we set the 'fetch-depth' input to 0
    # so we can see the complete git history.
    # Also, without this value checkout will not fetch tags (which we want).

    - name: Clone repository
      uses: actions/checkout@v4
      with:
        fetch-depth: 0

    # Now lets calculate the next version.
    # In this example we are still in prestable development phase.

    - name: Determine next version
      uses: tsdtsdtsd/nextversion-action@main 
      id: nextversion
      with:
        prestable-mode: true

    # Let's see what nextversion has to say

    - name: Debug nextversion output
      run: |
        echo 'hasCurrentVersion: ${{ steps.nextversion.outputs.hasCurrentVersion }}'
        echo 'hasNextVersion:    ${{ steps.nextversion.outputs.hasNextVersion }}'
        echo 'current:           ${{ steps.nextversion.outputs.currentVersion }}'
        echo 'current strict:    ${{ steps.nextversion.outputs.currentVersionStrict }}'
        echo 'next:              ${{ steps.nextversion.outputs.nextVersion }}'
        echo 'next strict:       ${{ steps.nextversion.outputs.nextVersionStrict }}'
        echo 'prestable:         ${{ steps.nextversion.outputs.prestableVersion }}'
        echo 'prestable strict:  ${{ steps.nextversion.outputs.prestableVersionStrict }}'
```

### Prestable Mode

From the semver specification:
> Major version zero (0.y.z) is for initial development. Anything MAY change at any time. The public API SHOULD NOT be considered stable.

And from the semver FAQ:
> **How should I deal with revisions in the 0.y.z initial development phase?**  
> The simplest thing to do is start your initial development release at 0.1.0 and then increment the minor version for each subsequent release.

While we're working on a `0.y.z` version and are not ready to "publish" it, we need to make sure that the automatic version bumping WILL NOT increment to `1.0.0`, even if there are conventional commits with a `BREAKING CHANGE` signal.

The only way I can think of to automate the process of jumping from `0.y.x` to `1.0.0` is to define additional in-house conventions to commit messages, e.g. if a commit message contains something like `(STABLE)`. I would like to not introduce such additional conventions.

The solution of nextversion-action is to control behaviour via action inputs.  
`prestable-mode` (which defaults to `false`) is the controlling flag here. As long as it is `true` and the newest git tag is `< v1.0.0`, nextversion-action will never increment major versions.

When you are done with prestable development and want to actually release a 1.0.0 you have two options:
- remove the `prestable-mode` input or set it to `false` in your workflow  
  currently you will also need to make sure that you have a breaking change signal in the commits since last version
- create and push an **annotated** git tag `v1.0.0` manually
