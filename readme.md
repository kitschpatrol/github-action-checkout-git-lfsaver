# Checkout Git LFS

Composite GitHub Action that checks out a repository and pulls its Git LFS objects from the custom LFS server configured in the repo's `.lfsconfig`.

## Why not `actions/checkout` with `lfs: true`?

`actions/checkout` runs `git lfs fetch` before the working tree exists, so a repo-local `.lfsconfig` pointing at a custom LFS server is never read — the fetch goes to `github.com/<repo>.git/info/lfs` and 404s. This action checks out first, then pulls LFS objects with `.lfsconfig` in place.

## Usage

```yaml
permissions:
  contents: read
  id-token: write # only needed for the OIDC fallback

steps:
  - uses: kitschpatrol/github-action-checkout-git-lfs-cf@v1
    with:
      lfs-pat: ${{ secrets.LFS_PAT }} # omit to use OIDC
```

## Inputs

| input        | default        | description                                                                                                            |
| ------------ | -------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `lfs-pat`    | _none_         | Token for the LFS server, sent as basic-auth password with username `pat`. When empty, falls back to OIDC (see below). |
| `repository` | current repo   | Repository to check out, `owner/name`.                                                                                 |
| `ref`        | triggering ref | Branch, tag, or SHA to check out.                                                                                      |

## Authentication

- **PAT** — set `lfs-pat`. Sent as basic auth (`pat:<token>`).
- **OIDC fallback** — when `lfs-pat` is empty, the action mints a GitHub Actions OIDC token with the LFS host as `audience` and sends it as basic auth (`oidc:<token>`). The calling job must grant `id-token: write`, and the LFS server must validate GitHub's OIDC issuer (`https://token.actions.githubusercontent.com`) and the expected audience.

The token is held only in the pull step's environment and passed to git via a per-invocation credential helper (`git -c`) — it is never written to `.git/config` or any other file.

## Requirements

- The target repo must have a `.lfsconfig` with `lfs.url` at its root (the action fails with a clear error otherwise).
- Runner needs `git-lfs`, `curl`, and `jq` (all preinstalled on GitHub-hosted runners).
