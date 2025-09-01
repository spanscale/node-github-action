# CI with Releases Pipeline Action for Node and PNPM

A complete CI with Releases pipeline for Node.js projects using PNPM, with optional release automation. While optimized for TypeScript projects, it works with any Node.js + PNPM project.

## Features

- Node.js and pnpm setup with configurable versions
- Customizable CI scripts (install, type-check, lint, test)
- Automatic semantic versioning based on conventional commits
- Changelog generation with categorized commit types
- GitHub release creation with prerelease support
- NPM package publishing
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
      - uses: spanscale/node-github-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Custom Node.js and pnpm Versions

```yaml
- uses: spanscale/node-github-action@v1
  with:
    node-version: '20.x'
    pnpm-version: '9'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Custom CI Scripts

```yaml
- uses: spanscale/node-github-action@v1
  with:
    run-scripts: '["pnpm install", "pnpm build", "pnpm test:unit", "pnpm test:integration"]'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### For Non-TypeScript Projects

```yaml
- uses: spanscale/node-github-action@v1
  with:
    run-scripts: '["pnpm install", "pnpm build", "pnpm test"]'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Working Directory

```yaml
- uses: spanscale/node-github-action@v1
  with:
    working-directory: './packages/api'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Custom NPM Registry

```yaml
- uses: spanscale/node-github-action@v1
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
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - uses: spanscale/node-github-action@v1
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
| `github-token` | GitHub token for authentication | Yes | - |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | The new version number if a release was created |
| `release-url` | URL of the created GitHub release |
| `package-url` | URL of the published package |

## Release Automation

### Conventional Commits

The action automatically determines version bumps based on conventional commit messages:

- `feat:` - Minor version bump
- `fix:` - Patch version bump
- `BREAKING CHANGE:` or `!:` - Major version bump
- `docs:`, `test:`, `chore:`, etc. - No version bump

### Special Release Tags

You can override automatic version detection using special tags in commit messages:

- `[release:stable]` - Force stable release based on conventional commits
- `[release:premajor]` - Premajor version bump
- `[release:preminor]` - Preminor version bump  
- `[release:prepatch]` - Prepatch version bump

### Manual Releases

Use workflow_dispatch to manually trigger releases with a specific version type:

```yaml
workflow_dispatch:
  inputs:
    release-type:
      type: choice
      options: [patch, minor, major, prepatch, preminor, premajor, prerelease]
```

## Examples

### CI Only

```yaml
- uses: spanscale/node-github-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### CI with Custom Scripts (Non-TypeScript)

```yaml
- uses: spanscale/node-github-action@v1
  with:
    run-scripts: '["pnpm install", "pnpm build", "pnpm test"]'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Full CI with Releases Pipeline

```yaml
- uses: spanscale/node-github-action@v1
  with:
    enable-release: ${{ github.ref == 'refs/heads/main' }}
    release-publish-package: 'true'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Requirements

- Repository must have a `package.json` file for version management
- For publishing, appropriate registry authentication must be configured
- For changelog generation, commit messages should follow conventional commit format
- GitHub token needs appropriate permissions for releases and package publishing

## License

MIT