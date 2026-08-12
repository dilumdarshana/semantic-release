# AGENTS.md

## What this repo is

A testing/demo project for semantic-release automation (publishes a trivial TS package to npm). Not a real library — keep changes small and aligned with the release pipeline.

## Release model (critical)

- Config lives in `.releaserc.json` (not `.github/semantic-release.yml`). Branches: `master` (stable) and `next` (prerelease, versions like `1.0.2-next.1`).
- Commit messages must follow Conventional Commits: `fix:` → patch, `feat:` → minor, `BREAKING CHANGE:` → major, `chore:`/`docs:` → **no release**. A non-conforming message silently produces no release.
- `package.json` `version` and `CHANGELOG.md` are **auto-managed** by semantic-release. Never hand-edit them or manually bump versions. The `@semantic-release/git` plugin auto-commits them with `chore(release): <version> [skip ci]` — that `[skip ci]` is what stops the CI workflow from re-triggering, so don't strip it.
- Workflow convention (README): create branches from `next`, merge back into `next`.

## Commands

- `pnpm build` — `tsc`, emits to `dist/` (gitignored). This is the **only** verification step; there are no tests, lint, or typecheck scripts.
- `pnpm semantic-release` — run the release (use `pnpm`, not `pnpx`: `pnpx` fetches the latest semantic-release from pnpm's cache instead of the lockfile-pinned version).
- Dry run: `pnpx semantic-release --debug --dry-run` (see README).
- Package manager is pnpm (`packageManager: pnpm@10.7.0`); don't use npm/yarn.

## CI (`.github/workflows/publish.yml`)

- Runs on push to `master` and `next`: `pnpm install && pnpm build && pnpx semantic-release`.
- `fetch-depth: 0` is required for semantic-release to see full history — don't remove it.
- Publishing to npm uses **trusted publishing (OIDC)** — the job must have `id-token: write`, and no `NPM_TOKEN` secret is used. Do not set `registry-url` in `actions/setup-node` (it creates an `.npmrc` that conflicts with OIDC auth). GitHub releases use the built-in `GITHUB_TOKEN` with `contents: write`.
