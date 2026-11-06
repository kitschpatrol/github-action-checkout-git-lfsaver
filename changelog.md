# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.2] - 2026-11-01

### Documentation

- Add a README badge linking to the latest GitHub release.

### Maintenance

- Refresh development tooling to `@kitschpatrol/shared-config` ^8.14.0 and pnpm 12.16.1, set the contributor Node.js requirement to `^24.16.0 || >=26.3.0`, and prune obsolete installation-policy exceptions.
- Update and pin CI, metadata, and release actions, and change dependency-update automation from weekly to monthly.
- Migrate development-skill installation to pnpm's native support, remove the `skills` dependency and prepare hook, and remove the `.claude/` cleanup exemption.
- Make link checks exclude loopback URLs and tolerate empty inputs.

## [1.0.1] - 2026-10-07

### Documentation

- Expand OIDC and token examples into complete workflows with triggers, permissions, jobs, and runner selection.

### Maintenance

- Refresh development tooling with `@kitschpatrol/shared-config` ^8.12.1, bumpp ^12.3.0, pnpm 12.9.1, updated lint configuration, and a pinned metadata-sync action.
- Add automatic development-skill synchronization during dependency installation.
- Add a weekly and manually triggered workflow for dependency-update pull requests.
- Improve website link checking with `lychee.toml` support, linked-sitemap discovery, and pinned crawler versions.
- Rename the contributor cleanup command from `clean` to `clean-deep`.

## [1.0.0] - 2026-08-02

### Added

- Provide a composite GitHub Action that checks out a selected repository and ref, then resolves LFS files using the custom server configured in its root `.lfsconfig`.
- Support GitHub PATs or server-issued credentials through `lfs-token`, falling back to host-audience OIDC with `id-token: write`, then anonymous downloads for public repositories.
- Validate `.lfsconfig` and HTTPS endpoints, require matching `lfs-host` pinning for supplied tokens, and keep authentication credentials out of Git configuration.

### Documentation

- Document `@v1` usage, authentication strategies and permissions, supported inputs, runner requirements, and when workflows need resolved LFS contents.

### Maintenance

- Set up shared linting, cross-platform CI, link checks, and automated GitHub releases with bumpp and moving major-version tags.

[1.0.0]: https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/releases/tag/v1.0.0
[1.0.1]: https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/compare/v1.0.0...v1.0.1
[1.0.2]: https://github.com/kitschpatrol/github-action-checkout-git-lfsaver/compare/v1.0.1...v1.0.2
