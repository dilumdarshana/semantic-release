# @orgme/semantic-release

A testing/demo project for **semantic-release** automation. It publishes a trivial TypeScript package to npm and demonstrates how commit messages drive automated versioning, changelog generation, and publishing.

## What is Semantic Release?

[Semantic Release](https://semantic-release.gitbook.io/semantic-release/) automates the entire release process — versioning, changelog generation, npm publishing, and GitHub releases — based entirely on your commit messages. You never manually bump versions or write release notes again.

## How It Works

1. You write commit messages following the [Conventional Commits](https://www.conventionalcommits.org/) format.
2. On every push to `master` or `next`, CI runs semantic-release.
3. Semantic-release analyzes the commits since the last release to determine the next version.
4. It updates `package.json` and `CHANGELOG.md`, publishes to npm, and creates a GitHub release — all automatically.

### Under the Hood: How Versions Are Resolved

Git tags are the **source of truth** for semantic-release. Each release creates a tag like `v1.0.2` (or `v1.0.2-next.1` for prereleases):

1. Semantic-release finds the **most recent tag** on the current branch.
2. It inspects all **commits between that tag and `HEAD`**.
3. Each conforming commit contributes a version bump; the **highest** bump wins:
   - a `fix:` → patch, a `feat:` → minor, a `BREAKING CHANGE:` → major.
4. The new version is written to `package.json`, appended to `CHANGELOG.md`, committed by the `@semantic-release/git` plugin, published to npm, and tagged as a GitHub release.

This means the `version` field in `package.json` is only a starting point — never edit it manually, semantic-release overwrites it.

## Key Features

- **Automated Versioning** — the next version is derived from commit messages (`fix` → patch, `feat` → minor, `BREAKING CHANGE` → major).
- **Changelog Generation** — `CHANGELOG.md` is regenerated automatically.
- **Publishing** — publishes the package to npm and creates a GitHub release.
- **Pre-releases** — the `next` branch produces prerelease versions (e.g. `1.0.2-next.1`).

## Prerequisites

- [Node.js](https://nodejs.org/) 22+
- [pnpm](https://pnpm.io/) 10.7.0 (this project uses pnpm — do not use npm/yarn)
- [GitHub CLI (`gh`)](https://cli.github.com/) — required for dry runs. Install it with `brew install gh`, then authenticate with `gh auth login`.
- An npm account — publishing uses **trusted publishing (OIDC)**, so no npm token is needed
- A GitHub repository with **trusted publishing** configured on npm for this package (map the repo + workflow file to the package at npmjs.com → package → Access)
- The `GITHUB_TOKEN` built-in secret (used to create GitHub releases and tags)

| Secret | Purpose |
| ------ | ------- |
| `GITHUB_TOKEN` | Built-in token used to create GitHub releases and tags |
| `id-token: write` | Job permission that lets GitHub mint OIDC tokens for npm publishing — no `NPM_TOKEN` required |

## Getting Started

### 1. Install dependencies

```bash
pnpm install
```

### 2. Build the project

```bash
pnpm build
```

Compiles TypeScript from `src/` to `dist/` (the `dist` folder is gitignored and is what gets published to npm).

### 3. Run a release

```bash
pnpm semantic-release
```

### 4. Dry run (safe to test)

Simulate a release without publishing anything:

```bash
GITHUB_TOKEN=$(gh auth token) pnpm release:dry-run
```

This is useful for verifying your configuration and seeing what version would be released. It requires the GitHub CLI (`gh`) to be installed and authenticated — otherwise the dry run fails. On macOS, install it with `brew install gh && gh auth login`.

## Commit Message Conventions

Semantic-release only releases when your commit messages follow the Conventional Commits format. The commit type determines the version bump:

| Commit type | Example | Version bump |
| ----------- | ------- | ------------ |
| `fix:` | `fix: correct minor typos in code` | Patch (`1.0.0` → `1.0.1`) |
| `feat:` | `feat: add user authentication feature` | Minor (`1.0.0` → `1.1.0`) |
| `BREAKING CHANGE:` | `BREAKING CHANGE: update API endpoint structure` | Major (`1.0.0` → `2.0.0`) |
| `chore:` | `chore: update dependencies` | **No release** |
| `docs:` | `docs: fix typo in README` | **No release** |

> **Note:** A commit message that does not conform to these conventions silently produces **no release**. Always use a recognized type.

## Development Workflow

### Doing a `next` (prerelease) release

The `next` branch is where all development happens. Pushing to it triggers a prerelease (e.g. `1.0.2-next.1`).

1. **Start from an up-to-date `next` branch:**

   ```bash
   git checkout next
   git pull
   ```

2. **Create a feature/fix branch from `next`:**

   ```bash
   git checkout -b fix/my-bug next
   ```

3. **Make your changes and commit them** with a proper Conventional Commits message:

   ```bash
   git add .
   git commit -m "fix: correct minor typos in code"
   ```

4. **Merge the branch back into `next`:**

   ```bash
   git checkout next
   git pull
   git merge fix/my-bug
   git push origin next
   ```

5. **CI runs automatically** — the push to `next` triggers the publish workflow, which produces a prerelease like `1.0.2-next.1` and publishes it to npm.

> **Tip:** If no release is needed, use `chore:` or `docs:` commit messages — they produce no release.

### Promoting `next` to a stable release

When the prerelease is ready for production, merge `next` into `master`. Pushing to `master` triggers the stable release (e.g. `1.0.2`).

```bash
git checkout master
git pull
git merge next
git push origin master
```

## Configuration

Release configuration lives in `.releaserc.json`. Here is this project's config, annotated:

```jsonc
{
  // Which branches trigger a release, and how.
  "branches": [
    "master",                    // Stable releases (e.g. 1.0.2)
    { "name": "next", "prerelease": true } // Prereleases (e.g. 1.0.2-next.1)
  ],
  "plugins": [
    // Analyzes commits since the last tag and determines the version bump.
    "@semantic-release/commit-analyzer",
    // Generates the release notes text from those commits.
    "@semantic-release/release-notes-generator",
    // Publishes the package to npm.
    "@semantic-release/npm",
    // Creates the GitHub release. Comments are disabled here.
    ["@semantic-release/github", { "successComment": false, "failComment": false }],
    // Writes the generated notes into CHANGELOG.md.
    ["@semantic-release/changelog", { "changelogFile": "CHANGELOG.md" }],
    // Commits the updated package.json + CHANGELOG.md. The "[skip ci]"
    // suffix prevents the CI workflow from re-triggering on this commit.
    ["@semantic-release/git", {
      "assets": ["package.json", "CHANGELOG.md"],
      "message": "chore(release): ${nextRelease.version} [skip ci]"
    }]
  ]
}
```

### Plugin execution order

Plugins run in sequence on every release. Understanding the order explains the whole pipeline:

| # | Plugin | What it does |
| - | ------ | ------------ |
| 1 | `commit-analyzer` | Determines the new version from commits |
| 2 | `release-notes-generator` | Writes the release notes |
| 3 | `npm` | Publishes to npm |
| 4 | `github` | Creates the GitHub release |
| 5 | `changelog` | Updates `CHANGELOG.md` |
| 6 | `git` | Commits `package.json` + `CHANGELOG.md` and tags the release |

> **Important:** `package.json` version and `CHANGELOG.md` are auto-managed by semantic-release. Never edit them by hand.

## CI/CD Pipeline

The workflow in `.github/workflows/publish.yml` runs on every push to `master` and `next`:

1. Checkout with full history (`fetch-depth: 0` — required for semantic-release).
2. Set up Node.js 22 and pnpm.
3. `pnpm install`
4. `pnpm build`
5. `pnpm semantic-release` (publishes to npm via **trusted publishing** and creates a GitHub release)

Publishing uses OIDC trusted publishing: the `release` job declares `id-token: write`, so GitHub mints an OIDC token that npm accepts — no `NPM_TOKEN` secret, and npm **provenance** is generated automatically for public packages. Note the trusted publisher on npmjs.com must reference this exact workflow file (`publish.yml`). Use `pnpm semantic-release` (not `pnpx`) so the release runs the lockfile-pinned versions instead of fetching the latest from pnpm's cache.

## Troubleshooting

| Symptom | Cause | Fix |
| ------- | ----- | --- |
| No release created despite a push | Commit message doesn't conform to Conventional Commits (or only `chore:`/`docs:` commits) | Use `fix:`, `feat:`, or include `BREAKING CHANGE:` |
| Dry run fails with a token/GitHub error | `gh` CLI not installed or not authenticated | `brew install gh && gh auth login` |
| "This command requires a token" | `GITHUB_TOKEN` env var not set | Run `GITHUB_TOKEN=$(gh auth token) pnpm release:dry-run` |
| npm publish fails | `NPM_TOKEN` secret missing or invalid | Add the token to the repo's GitHub secrets |
| semantic-release can't see past releases | Shallow checkout | Keep `fetch-depth: 0` in the checkout step |
| "Tag already exists" on a rerun | CI re-triggered on the release commit | Don't strip `[skip ci]` from the release commit message |
| Version not bumped as expected | Pre-release/branch logic — `next` always produces `-next.x`, only `master` goes stable | Check which branch you pushed to |

## Resources

- [Semantic Release Documentation](https://semantic-release.gitbook.io/semantic-release/)
- [Commit Message Guidelines](https://www.conventionalcommits.org/)