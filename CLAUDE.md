# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a reusable GitHub Action that provides a complete CI/CD pipeline for Node.js projects using PNPM. While optimized for TypeScript projects, it works with any Node.js + PNPM project. The action is designed to be flexible and configurable, supporting:

- Node.js and pnpm setup
- Configurable CI scripts (install, type-check, lint, test)
- Optional automated release workflow with semantic versioning
- Changelog generation using conventional commits
- Package publishing to NPM registries
- Issue commenting for releases

## Architecture

The entire CI/CD pipeline is contained in a single `action.yml` file that defines:

- **Inputs**: 20+ configurable parameters for customization
- **Composite Action Steps**: Sequential steps for setup, CI, and release
- **Version Determination**: Automatic semantic version bumping based on commit messages or manual triggers
- **Changelog Generation**: Categorized changelog based on conventional commit types
- **Release Automation**: GitHub release creation and optional NPM publishing

## Key Components

### Script Execution with Fallback (action.yml:95-118)
- Parses `run-scripts` input as JSON array
- Includes fallback mechanism when GitHub Actions doesn't pass default values properly
- Default scripts: `["pnpm install --frozen-lockfile", "pnpm type-check", "pnpm lint", "pnpm build", "pnpm test"]`
- Debug output helps troubleshoot script parsing issues

### Version Bumping Logic (action.yml:127-198)
- Supports conventional commits (`feat:`, `fix:`, `BREAKING CHANGE`)
- Manual release types via workflow_dispatch
- Prerelease support with `-beta` suffix
- Special release tags: `[release:stable]`, `[release:premajor]`, etc.

### Changelog Generation (action.yml:199-275)
- Categorizes commits by type (feat, fix, perf, refactor, docs, test, chore, ci, style)
- Generates structured markdown changelog
- Links to full changelog on GitHub

### Release Workflow (action.yml:276-340)
- Commits version bump and changelog
- Creates GitHub releases (with prerelease detection)
- Optional NPM publishing
- Issue commenting with release links

## Examples Directory

The `examples/` directory contains real-world usage patterns:
- **basic-ci.yml**: Simple CI pipeline with default settings
- **library-release.yml**: Complete CI/CD with NPM publishing
- **monorepo-ci.yml**: Multi-package testing with working directories
- **github-packages.yml**: Publishing to GitHub Package Registry
- **prerelease-workflow.yml**: Beta/prerelease automation
- **custom-scripts.yml**: Custom build tools and testing frameworks

## Critical Requirements for Consumers

### Required Workflow Setup
1. **Checkout Step**: Must include `actions/checkout@v4` before using this action
2. **Fetch Depth**: Use `fetch-depth: 0` for release workflows to get full git history
3. **Permissions**: For releases, set appropriate workflow permissions:
   ```yaml
   permissions:
     contents: write    # For creating releases and tags
     packages: write    # For publishing packages
     issues: write      # For commenting on issues
   ```

### Common Issues and Solutions
- **`ERR_PNPM_NO_PKG_MANIFEST`**: Missing checkout step in consumer workflow
- **`pnpm` command without arguments**: GitHub Actions not passing default input values (fixed with fallback logic)
- **Release not triggering**: Ensure `enable-release: ${{ github.ref == 'refs/heads/main' }}` for main branch only

## Development and Maintenance

### Versioning Workflow
1. Make changes to `action.yml`
2. Test changes with example workflows
3. Commit with conventional commit messages
4. Tag new versions: `git tag -a v1.x.x -m "description"`
5. Push tags: `git push origin v1.x.x`

### Testing Changes
- No traditional test suite - this is action metadata only
- Test by running example workflows against modified action
- Validate `action.yml` syntax with GitHub Actions schema
- Ensure backward compatibility for existing consumers

### Configuration Defaults
- Default: pnpm with Node.js 22.x, GitHub Package Registry
- CI Scripts fallback ensures action works even when defaults aren't passed
- Release features are opt-in via `enable-release: true`
- Works with any Node.js + PNPM project by customizing the `run-scripts` input