# Examples

This directory contains real-world examples of using the TypeScript CI/CD Pipeline Action in different scenarios.

## Available Examples

### [basic-ci.yml](./basic-ci.yml)
Simple CI pipeline that runs on every push and pull request with default settings.
- Uses default Node.js 22.x and pnpm 10
- Runs standard scripts: install, type-check, lint, test
- No release automation

### [monorepo-ci.yml](./monorepo-ci.yml)
CI pipeline for monorepo projects with multiple packages.
- Tests multiple packages independently
- Uses `working-directory` to target specific packages
- Custom scripts for different package types (API, web, utils)
- Integration and E2E testing examples

### [library-release.yml](./library-release.yml)
Complete CI/CD pipeline for npm library with automated releases.
- Custom Node.js and pnpm versions
- NPM registry publishing
- Automated semantic versioning
- Changelog generation
- Issue commenting
- Manual release triggers

### [github-packages.yml](./github-packages.yml)
CI/CD pipeline publishing to GitHub Packages Registry.
- GitHub Packages configuration
- Security auditing
- Automated releases to GitHub registry
- Proper permissions setup

### [nextjs-app.yml](./nextjs-app.yml)
Multi-environment deployment for Next.js applications.
- Staging and production deployments
- Custom Next.js build scripts
- E2E testing with headless browsers
- Environment-specific configurations

### [prerelease-workflow.yml](./prerelease-workflow.yml)
Automated prerelease creation from develop branch.
- Beta/prerelease versioning
- Enhanced testing for prereleases
- Team notifications
- Manual prerelease triggers

### [custom-scripts.yml](./custom-scripts.yml)
Examples using custom build tools and testing frameworks.
- Vite and Vitest setup
- Webpack configuration
- Security-focused pipeline with auditing
- Custom bundler and analyzer tools

## Usage

1. Copy the relevant example to your `.github/workflows/` directory
2. Modify the configuration to match your project structure
3. Update script names to match your package.json scripts
4. Configure secrets and permissions as needed
5. Adjust trigger conditions for your workflow

## Common Configurations

### Registry Setup
- **NPM**: Use `registry-url: 'https://registry.npmjs.org'` and set `NODE_AUTH_TOKEN` secret
- **GitHub Packages**: Use default settings and ensure proper repository permissions

### Permissions
For releases and publishing, ensure your workflow has:
```yaml
permissions:
  contents: write    # For creating releases and tags
  packages: write    # For publishing packages
  issues: write      # For commenting on issues
```

### Secrets
Common secrets you may need:
- `GITHUB_TOKEN`: Usually available by default
- `NPM_TOKEN`: For NPM registry publishing
- Custom deployment tokens for your platforms

## Tips

- Use `fetch-depth: 0` when enabling releases to get full git history
- Set `enable-release` conditionally based on branch: `${{ github.ref == 'refs/heads/main' }}`
- Customize `run-scripts` to match your project's npm scripts
- Use `working-directory` for monorepos or nested project structures
- Consider security implications when publishing packages automatically