# kitschpatrol/github-action-checkout-git-lfsaver

<!-- badges -->

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/license/mit)
[![CI](https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/actions/workflows/ci.yml/badge.svg)](https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/actions/workflows/ci.yml)

<!-- /badges -->

<!-- description -->

**GitHub Action to check out repos and smudge Git LFS files hosted by a git-lfsaver server.**

<!-- /description -->

This action replaces [`actions/checkout`](https://github.com/actions/checkout) in workflows that need the smudged (resolved) contents of Git LFS files stored on a [Git LFSaver](https://github.com/kitschpatrol/git-lfsaver) server. It checks out the repository, then pulls LFS objects from the server named in the repo's committed `.lfsconfig` — authenticating with a GitHub OIDC token, a provided token, or anonymously for public repos.

If your workflow only needs the LFS pointer files — often the case — plain `actions/checkout` works as-is and this action isn't necessary.

## Why not `actions/checkout` with `lfs: true`?

The `lfs` input on `actions/checkout` runs the LFS fetch before the working tree exists, so a repo-local `.lfsconfig` pointing at a custom LFS server is never read, and the fetch 404s against `github.com/<repo>.git/info/lfs`. This action checks out first, authenticates, and then pulls LFS objects once the `.lfsconfig` is in place.

## Usage

With OIDC authentication (recommended):

```yaml
on:
  push: {}

permissions:
  contents: read
  id-token: write

jobs:
  lfs-check:
    runs-on: ubuntu-latest
    steps:
      - uses: kitschpatrol/github-action-checkout-git-lfsaver@v1
```

With a token, pinning the LFS host:

```yaml
on:
  push: {}

permissions:
  contents: read

jobs:
  lfs-check:
    runs-on: ubuntu-latest
    steps:
      - uses: kitschpatrol/github-action-checkout-git-lfsaver@v1
        with:
          lfs-token: ${{ secrets.LFS_TOKEN }}
          lfs-host: lfs.example.com
```

## Requirements

- The checked-out repo must have a `.lfsconfig` at its root with `lfs.url` pointing to a [Git LFSaver](https://github.com/kitschpatrol/git-lfsaver) server instance. The action fails with a clear error otherwise.
- The runner needs `git-lfs`, `curl`, and `jq`, which are all preinstalled on GitHub-hosted Linux, macOS, and Windows runners.

## Inputs

| input        | default        | description                                                                                                                                           |
| ------------ | -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `lfs-token`  | _none_         | Credential for the LFS server: a GitHub personal access token or a self-issued token. When empty, falls back to OIDC, then anonymous (see below).     |
| `lfs-host`   | _none_         | Expected LFS server hostname, e.g. `lfs.example.com`. Required with `lfs-token`; the action refuses to authenticate if `.lfsconfig` points elsewhere. |
| `repository` | current repo   | Repository to check out, `owner/name`.                                                                                                                |
| `ref`        | triggering ref | Branch, tag, or SHA to check out. When `repository` is another repo, defaults to that repo's default branch instead.                                  |

Only the `repository` and `ref` inputs from `actions/checkout` are supported. If you need others (`fetch-depth`, `submodules`, `path`, …), [open an issue](https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/issues).

## Authentication

The action picks one of three strategies, in order:

### Token (`lfs-token`)

When `lfs-token` is set, it's sent as the Basic auth password. The server detects the credential type from its shape, so this works with a GitHub personal access token (fine-grained tokens can be scoped to a single repo) or a [self-issued token](https://github.com/kitschpatrol/git-lfsaver#self-issued-tokens) minted by the server operator.

Why `lfs-host` is required with a token: the transfer host is read from the checked-out `.lfsconfig`, so on an untrusted ref a tampered file could redirect the credential to an attacker's server. Pinning the hostname in the workflow — which the checked-out ref can't modify — makes the action refuse the mismatch instead.

### OIDC

When `lfs-token` is empty and the job grants `id-token: write`, the action mints a short-lived GitHub Actions OIDC token with the LFS host as `audience` and sends it as the Basic auth password. No secrets are stored, and the server verifies the token against GitHub's public keys — its cryptographically verified `repository` claim must match the repo in the LFS URL, and it's download-only by design. No `lfs-host` pin is needed: OIDC tokens are audience-bound to the host they were minted for and useless elsewhere.

The token is held only in the pull step's environment and passed to git via a per-invocation credential helper (`git -c`) — it is never written to `.git/config` or any other file.

### Anonymous (public repos)

With no `lfs-token` and no OIDC token available, the action pulls without a credential, which the server allows for downloads from public GitHub repos. This is what keeps fork PRs working: forks never receive `id-token: write`, so on public repos the action falls back to an anonymous pull instead of failing.

<!-- license -->

## License

[MIT](license.txt) © [Eric Mika](https://ericmika.com)

<!-- /license -->
