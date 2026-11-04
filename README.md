# workflows

Reusable GitHub Actions workflows for AYÇİÇEK Labs projects. Simple standards repositories do not use them.

| Workflow           | What it checks                                                                                                                                                                          |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pull-request.yml` | On pull requests: every commit and the title follow [`commitlint`](https://github.com/AYCICEK-Labs/commitlint); the description keeps every template section and has no AI attribution. |
| `docs.yml`         | Every document in `docs/` has an `.en.md` and a `.tr.md` file, and a pull request changes both.                                                                                         |
| `format.yml`       | Every file follows the project's `.prettierrc` from [`prettier`](https://github.com/AYCICEK-Labs/prettier).                                                                             |
| `node.yml`         | Node projects: version from `.nvmrc`, `npm ci`, then `lint`, `test` and `build` when the scripts exist.                                                                                 |
| `rust.yml`         | Rust projects: toolchain from `rust-toolchain.toml`, then `cargo fmt`, Clippy with warnings as errors, and tests.                                                                       |
| `release.yml`      | Optional: [release-please](https://github.com/googleapis/release-please) keeps a release pull request with the next version and `CHANGELOG.md`.                                         |

## Versions

Projects use the `v1` tag. Compatible changes move `v1`; a change that can break a project gets `v2`.

## Use it in a project

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, edited, synchronize, reopened]

permissions:
  contents: read

jobs:
  pull-request:
    if: github.event_name == 'pull_request'
    uses: AYCICEK-Labs/workflows/.github/workflows/pull-request.yml@v1

  docs:
    uses: AYCICEK-Labs/workflows/.github/workflows/docs.yml@v1

  format:
    uses: AYCICEK-Labs/workflows/.github/workflows/format.yml@v1

  node:
    uses: AYCICEK-Labs/workflows/.github/workflows/node.yml@v1
```

Leave out `node` in projects without a `package.json`. Rust projects use `rust` instead:

```yaml
rust:
  uses: AYCICEK-Labs/workflows/.github/workflows/rust.yml@v1
```

A Rust project with private Git dependencies passes a token that can read them, as `secrets: { cargo_git_token: ${{ secrets.NAME }} }`.

`edited` runs the checks again when the pull request title or description changes.

### Releases

`.github/workflows/release.yml`:

```yaml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    uses: AYCICEK-Labs/workflows/.github/workflows/release.yml@v1
    with:
      release-type: node
    secrets: inherit
```

A pull request opened with the default token does not start other workflows, so CI does not run on the release pull request. To fix that, add a fine-grained token with **Contents** and **Pull requests** read and write access as the `RELEASE_PLEASE_TOKEN` secret.

## License

[MIT](LICENSE) © Yunus Emre AYÇİÇEK
