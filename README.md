# Testing Semantic Release

Semantic Release automates the versioning and package publishing process based on commit messages. This ensures consistent releases and changelogs.

## Dry Run
Use the following command to simulate a release without actually publishing:
```bash
pnpx semantic-release --debug --dry-run
```

## Setup Environment Variables
To enable publishing, create an access token on the npm site and add it to GitHub as a secret.

## Steps to Use Semantic Release

1. **Install Semantic Release**:
   ```bash
   pnpm install semantic-release --save-dev
   ```

2. **Run Semantic Release**:
   ```bash
   pnpx semantic-release
   ```

## Key Features
- **Automated Versioning**: Determines the next version based on commit messages.
- **Changelog Generation**: Automatically updates the changelog.
- **Publishing**: Publishes the package to npm and creates a GitHub release.

## Example Commit Messages
- `fix: Correct minor typos in code` (patch release)
- `feat: Add new user authentication feature` (minor release)
- `BREAKING CHANGE: Update API endpoint structure` (major release)
- `chore: Maintenance or internal tooling updates that don't affect the app’s behavior` (no release)
  - Example: `chore: update dependencies`
  - Example: `chore: rename folder structure`
- `docs: Updates to content that does not affect the source code logic or application behavior` (no release)
  - Example: `docs: fix typo in README`
  - Example: `docs: add usage examples for CLI`

## Development Workflow
- Always create a branch from the `next` branch for any fixes or features.
- Merge changes back into the `next` branch with a proper commit message.
- If no release is required, use generic commit messages starting with `chore`, `docs`, etc.

## Resources
- [Semantic Release Documentation](https://semantic-release.gitbook.io/semantic-release/)
- [Commit Message Guidelines](https://www.conventionalcommits.org/)
