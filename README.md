# workflows

Reusable GitHub Actions workflows shared by every FuckAPK project repository.

## `build.yml`

Call it from a project repository:

```yaml
name: Build

on:
  push:
    branches:
      - main
    tags:
      - '*'
  # NOT `pull_request_target`: that event runs in the context of the *base* branch, so
  # `actions/checkout` builds `main` instead of the pull request.
  pull_request:
  workflow_dispatch:

jobs:
  build:
    uses: FuckAPK/workflows/.github/workflows/build.yml@v1
    permissions:
      contents: write
    secrets: inherit
```

### Jobs

| job | when | needs secrets | what it does |
|---|---|---|---|
| `verify` | always | no | `./gradlew assembleDebug` - this is what gates pull requests |
| `package` | `push` / `workflow_dispatch` | yes | signs and assembles the release APK, uploads it as an artifact, and attaches it to a GitHub release on tag pushes |

### Why it is split this way

* Pull requests must not need signing secrets. Workflows triggered by a Dependabot pull
  request run as if they came from a fork: the token is read-only and **no repository
  secrets are available**. When the release signing step lived in the same job as the
  verification build, every Dependabot pull request failed for infrastructure reasons and
  dependency updates could never be validated.
* The caller must use `pull_request`, not `pull_request_target`. With
  `pull_request_target`, `actions/checkout` checks out the base branch, so the build
  validates `main` and the green check says nothing about the pull request.
* The signing secrets are declared `required: false` so that the `verify` job can run
  without them.

### Versioning of this repository

Project repositories reference `@v1`. Bump the tag when a change to these workflows should
reach the projects; a floating `@main` reference would silently change ten repositories at
once.
