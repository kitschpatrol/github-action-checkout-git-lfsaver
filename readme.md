# kitschpatrol/github-action-checkout-git-lfsaver

<!-- badges -->

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/license/mit)
[![CI](https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/actions/workflows/ci.yml/badge.svg)](https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/actions/workflows/ci.yml)

<!-- /badges -->

<!-- description -->

**GitHub Action to check out repos and smudge Git LFS files hosted by a git-lfsaver server.**

<!-- /description -->

This action is a drop-in replacement for [`actions/checkout`](https://github.com/actions/checkout) for CI and other GitHub Action workflows in repositories using the [Git LFSaver](https://github.com/kitschpatrol/git-lfsaver) server to store large files.

It's only necessary if your workflow needs access to smudged (resolved) LFS file objects. In many cases, just checking out pointers is fine, and for this `actions/checkout` continues to work fine.

GitHub Action that checks out a repository and pulls its Git LFS objects from the [GIT LFSaver](https://github.com/kitschpatrol/git-lfsaver) server configured in the repo's `.lfsconfig`.

## Why not `actions/checkout` with `lfs: true`?

Note that even though `actions/checkout` has a `lfs` input options, this runs `git lfs fetch` before the working tree exists, so a repo-local `.lfsconfig` pointing at a custom LFS server is never read and the fetch will 404 at `github.com/<repo>.git/info/lfs`. This action checks out first, authenticates, and then pulls LFS objects once the `.lfsconfig` is in place.

## Usage

To use OIDC authentication (recommended):

```yaml
permissions:
  contents: read
  id-token: write

steps:
  - uses: kitschpatrol/github-action-checkout-git-lfsaver@v1
```

## Requirements

- The target repo must have a `.lfsconfig` with `lfs.url` at its root pointing to a [Git LFSaver](https://github.com/kitschpatrol/git-lfsaver) server instance. The action will fails with a clear error otherwise.
- The runner needs `git-lfs`, `curl`, and `jq`, which are all preinstalled on GitHub-hosted runners.

## Inputs

| input        | default        | description                                                                                                              |
| ------------ | -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `lfs-pat`    | _none_         | Token for the LFS server, sent as basic-auth password with username `pat`. When empty, falls back to OIDC (see below).   |
| `lfs-host`   | _none_         | Expected LFS server hostname, e.g. `lfs.example.com`. Required with `lfs-pat`; refused if `.lfsconfig` points elsewhere. |
| `repository` | current repo   | Repository to check out, `owner/name`.                                                                                   |
| `ref`        | triggering ref | Branch, tag, or SHA to check out. When `repository` is another repo, defaults to that repo's default branch instead.     |

## Authentication

### OIDC

When `lfs-pat` is empty, the action mints a GitHub Actions OIDC token with the LFS host as `audience` and sends it as basic auth (`oidc:<token>`). The calling job must grant `id-token: write`, and the LFS server must validate GitHub's OIDC issuer (`https://token.actions.githubusercontent.com`) and the expected audience.

The token is held only in the pull step's environment and passed to git via a per-invocation credential helper (`git -c`) — it is never written to `.git/config` or any other file.

### PAT

Set `lfs-pat`. Sent as basic auth (`pat:<token>`).

Why `lfs-host` is required with a PAT: the transfer host is read from the checked-out `.lfsconfig`, so on an untrusted ref a tampered file could redirect the credential to an attacker's server. Pinning the hostname in the workflow — which the checked-out ref can't modify — makes the action refuse the mismatch instead. OIDC needs no pin: its tokens are audience-bound to the host they were minted for and useless anywhere else.
