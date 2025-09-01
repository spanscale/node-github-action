# CI with Releases Pipeline Action for Node and PNPM

A complete CI with Releases pipeline for Node.js projects using PNPM, with optional release automation. While optimized for TypeScript projects, it works with any Node.js + PNPM project.

## Features

- Node.js and pnpm setup with configurable versions
- Customizable CI scripts (install, type-check, lint, test)
- **Advanced versioning control** with 20+ built-in patterns
- **PR title pattern detection** (enabled by default)
- Automatic semantic versioning based on conventional commits
- **Custom version patterns** and **exact version control**
- **First commit bump control** and **skip rules**
- Changelog generation with categorized commit types
- GitHub release creation with prerelease support
- **Configurable git tagging**
- NPM package publishing with automatic authentication
- Issue commenting for releases

## Usage

### Basic CI Pipeline

```yaml
name: CI
on: [push, pull_request]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: spanscale/node-github-action@v1.0.0
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Custom Node.js and pnpm Versions

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    node-version: '20.x'
    pnpm-version: '9'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Custom CI Scripts

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    run-scripts: '["pnpm install", "pnpm build", "pnpm test:unit", "pnpm test:integration"]'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### For Non-TypeScript Projects

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    run-scripts: '["pnpm install", "pnpm build", "pnpm test"]'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Working Directory

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    working-directory: './packages/api'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Custom NPM Registry

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    registry-url: 'https://registry.npmjs.org'
    registry-scope: '@mycompany'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Complete CI with Releases

```yaml
name: CI with Releases
on: 
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      release-type:
        description: 'Release type'
        required: true
        default: 'patch'
        type: choice
        options:
        - patch
        - minor
        - major
        - prepatch
        - preminor
        - premajor
        - prerelease

jobs:
  ci-cd:
    runs-on: ubuntu-latest
    permissions:
      contents: write    # For creating releases and pushing tags
      packages: write    # For publishing packages
      issues: write      # For commenting on issues
      pull-requests: read # For reading PR information
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - uses: spanscale/node-github-action@v1.0.0
        with:
          enable-release: ${{ github.ref == 'refs/heads/main' }}
          release-bump-version: 'true'
          release-create-changelog: 'true'
          release-publish-package: 'true'
          release-comment-issues: 'true'
          release-type: ${{ github.event.inputs.release-type || 'patch' }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `node-version` | Node.js version to use | No | `22.x` |
| `pnpm-version` | pnpm version to use | No | `10` |
| `working-directory` | Working directory for commands | No | `.` |
| `run-scripts` | Scripts to run (JSON array) | No | `["pnpm install --frozen-lockfile", "pnpm type-check", "pnpm lint", "pnpm test"]` |
| `registry-url` | NPM registry URL | No | `https://npm.pkg.github.com` |
| `registry-scope` | NPM registry scope | No | `@spanscale` |
| `enable-release` | Enable release workflow after CI passes | No | `false` |
| `release-bump-version` | Enable automatic version bumping | No | `true` |
| `release-create-changelog` | Enable automatic changelog generation | No | `true` |
| `release-comment-issues` | Enable commenting on related issues | No | `false` |
| `release-publish-package` | Enable package publishing | No | `false` |
| `release-type` | Release type for manual triggers | No | `patch` |
| `release-skip-rules` | Skip release if commit/PR matches patterns (JSON array) | No | `["[skip ci]", "[skip release]", "chore(release):"]` |
| `release-first-commit-bump` | Enable version bump on first commit | No | `false` |
| `release-version-patterns` | Custom patterns for version determination (JSON object) | No | 20+ built-in patterns |
| `release-create-tag` | Create git tag for the release | No | `true` |
| `release-use-pr-patterns` | Use PR title for pattern detection | No | `true` |
| `release-override-existing` | Override existing tags/releases if they exist | No | `false` |
| `package-name` | Package name for publishing (auto-detected from package.json if not specified) | No | - |
| `github-token` | GitHub token for authentication | Yes | - |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The new version number if a release was created |
| `release-url` | URL of the created GitHub release |
| `package-url` | URL of the published package |

## Release Automation

The action supports advanced version control with 20+ built-in patterns, PR title detection, custom patterns, and more.

**📖 For complete versioning documentation, see [VERSIONING.md](./VERSIONING.md)**

### Handling Existing Releases

By default, the action will fail if a tag or release already exists for the version being released. You can control this behavior with the `release-override-existing` parameter:

- `false` (default): Fail the workflow if tag/release already exists
- `true`: Delete the existing tag/release and create a new one

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    enable-release: true
    release-override-existing: true  # Allow overriding existing releases
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Quick Examples

**Using built-in patterns:**
- `[beta]` - 1.0.0 → 1.0.1-beta.0
- `[hotfix]` - Patch release
- `[major]` - Major version bump

**PR title patterns (automatic):**
- PR title: `feat: Add auth [feature]` → Minor bump
- PR title: `fix: Bug fix [beta]` → 1.0.0 → 1.0.1-beta.0

**Conventional commits (fallback):**
- `feat:` - Minor version bump
- `fix:` - Patch version bump
- `BREAKING CHANGE:` - Major version bump

## Examples

### CI Only

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### CI with Custom Scripts (Non-TypeScript)

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    run-scripts: '["pnpm install", "pnpm build", "pnpm test"]'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Full CI with Releases Pipeline

```yaml
jobs:
  ci-cd:
    runs-on: ubuntu-latest
    permissions:
      contents: write    # For creating releases and pushing tags
      packages: write    # For publishing packages
      issues: write      # For commenting on issues
      pull-requests: read # For reading PR information
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - uses: spanscale/node-github-action@v1.0.0
        with:
          enable-release: ${{ github.ref == 'refs/heads/main' }}
          release-publish-package: 'true'
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Requirements

- Repository must have a `package.json` file for version management
- For publishing, appropriate registry authentication must be configured
- For changelog generation, commit messages should follow conventional commit format
- **GitHub token needs appropriate permissions for releases and package publishing**

### Required Permissions

When using release features, your workflow **must** include these permissions:

```yaml
permissions:
  contents: write    # Required for creating releases and pushing tags
  packages: write    # Required for publishing packages (if release-publish-package: true)
  issues: write      # Required for issue commenting (if release-comment-issues: true)
  pull-requests: read # Required for PR title pattern detection
```

### Common Permission Errors

**❌ `403 Permission permission_denied: write_package`**
- Missing `packages: write` permission in workflow
- Add the permissions section to your workflow job

**❌ `403 Resource not accessible by integration`**
- Missing `contents: write` permission in workflow
- Required for creating releases and pushing tags

**❌ `403 Permission permission_denied: write_package` (persists after adding permissions)**
- Package name in `package.json` doesn't match repository owner/organization
- For GitHub Packages, package name must be scoped to repository owner
- Example: Repository `myorg/mypackage` requires package name `@myorg/mypackage`
- See [Package Name Handling](#package-name-handling) below for solutions

### Package Name Handling

The action automatically detects your package name from `package.json`. If your repository name differs from your desired package name, you have several options:

**Option 1: Update package.json (Recommended for GitHub Packages)**
```json
{
  "name": "@myorg/my-awesome-package",
  "version": "1.0.0"
}
```

**Option 2: Override package name in workflow**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    package-name: "@myorg/my-awesome-package"
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

**Option 3: Use different registry (for public packages)**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    registry-url: 'https://registry.npmjs.org'
    registry-scope: ''  # No scope needed for npmjs.org
    github-token: ${{ secrets.GITHUB_TOKEN }}
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## License

MIT