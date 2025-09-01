# Versioning Guide

This guide explains how to control automatic versioning in your CI/CD pipeline using commit message patterns and configuration options.

## Quick Start

### Default Behavior (Zero Configuration)

Just use the action with `enable-release: true`:

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    enable-release: true
```

**What happens:**
- `feat: add login` → Minor version bump (0.1.0 → 0.2.0)
- `fix: login bug` → Patch version bump (0.1.0 → 0.1.1) 
- `BREAKING CHANGE: remove API` → Major version bump (0.1.0 → 1.0.0)
- `docs: update readme` → No version bump

## Understanding Version Bumps

### Semantic Versioning (SemVer)
Versions follow `MAJOR.MINOR.PATCH` format:
- **MAJOR**: Breaking changes (1.0.0 → 2.0.0)
- **MINOR**: New features, backward compatible (1.0.0 → 1.1.0)  
- **PATCH**: Bug fixes, backward compatible (1.0.0 → 1.0.1)
- **PRERELEASE**: Beta/alpha versions (1.0.0 → 1.0.1-beta.0)

### Built-in Pattern Support

The action includes comprehensive built-in patterns that work out of the box:

#### Standard Release Patterns
| Pattern | Version Bump | Example |
|---------|--------------|---------|
| `[major]` | Major | 1.0.0 → 2.0.0 |
| `[minor]` | Minor | 1.0.0 → 1.1.0 |
| `[patch]` | Patch | 1.0.0 → 1.0.1 |
| `[hotfix]` | Patch | 1.0.0 → 1.0.1 |
| `[urgent]` | Patch | 1.0.0 → 1.0.1 |
| `[security]` | Patch | 1.0.0 → 1.0.1 |
| `[breaking]` | Major | 1.0.0 → 2.0.0 |
| `[feature]` | Minor | 1.0.0 → 1.1.0 |

#### Prerelease Patterns
| Pattern | Version Bump | Example |
|---------|--------------|---------|
| `[beta]` | Prepatch | 1.0.0 → 1.0.1-beta.0 |
| `[alpha]` | Preminor | 1.0.0 → 1.1.0-beta.0 |
| `[rc]` | Prerelease | 1.0.0-beta.0 → 1.0.0-beta.1 |
| `[stable]` | Patch | 1.0.0-beta.0 → 1.0.0 |

#### Advanced Release Patterns  
| Pattern | Version Bump | Example |
|---------|--------------|---------|
| `[release:major]` | Major | 1.0.0 → 2.0.0 |
| `[release:minor]` | Minor | 1.0.0 → 1.1.0 |
| `[release:patch]` | Patch | 1.0.0 → 1.0.1 |
| `[release:prepatch]` | Prepatch | 1.0.0 → 1.0.1-beta.0 |
| `[release:preminor]` | Preminor | 1.0.0 → 1.1.0-beta.0 |
| `[release:premajor]` | Premajor | 1.0.0 → 2.0.0-beta.0 |
| `[release:prerelease]` | Prerelease | 1.0.0-beta.0 → 1.0.0-beta.1 |
| `[release:v2.1.0]` | Exact | Any → 2.1.0 |
| `[release:stable]` | Patch | 1.0.0-beta.0 → 1.0.0 |

### Conventional Commits (Fallback)
If no patterns match, conventional commits are used:

| Commit Message | Version Bump | Example |
|----------------|--------------|---------|
| `feat: new feature` | Minor | 1.0.0 → 1.1.0 |
| `fix: bug fix` | Patch | 1.0.0 → 1.0.1 |
| `feat!: breaking change` | Major | 1.0.0 → 2.0.0 |
| `BREAKING CHANGE:` | Major | 1.0.0 → 2.0.0 |
| `docs:`, `test:`, `chore:` | None | No bump |

## Common Use Cases

### 1. First Release Setup

**Problem:** Don't want `feat: initial release` to bump from 0.1.0 → 0.2.0

**Solution:**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    enable-release: true
    release-first-commit-bump: false  # Skip bump on first commit
```

**Alternative:** Use non-feature commit message:
```bash
git commit -m "chore: initial release"  # No version bump
```

### 2. Hotfix Releases

**Goal:** Quickly release patch fixes with custom patterns

**Setup:**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    enable-release: true
    release-version-patterns: |
      {
        "[hotfix]": "patch",
        "[urgent]": "patch"
      }
```

**Usage:**
```bash
git commit -m "any message [hotfix]"  # Always patch bump
git commit -m "urgent fix [urgent]"   # Always patch bump
```

### 3. Exact Version Control

**Goal:** Set specific version numbers

**Setup:**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    enable-release: true
    release-version-patterns: |
      {
        "[release:v*]": "exact"
      }
```

**Usage:**
```bash
git commit -m "feat: major rewrite [release:v2.0.0]"  # Sets exactly 2.0.0
git commit -m "fix: critical bug [release:v1.2.3]"   # Sets exactly 1.2.3
```

### 4. Beta/Prerelease Versions

**Goal:** Create beta versions for testing

**Simple Built-in Patterns (Recommended):**
```bash
# These patterns work out of the box with zero configuration
git commit -m "feat: new feature [beta]"    # 1.0.0 → 1.0.1-beta.0
git commit -m "fix: bug fix [beta]"          # 1.0.0 → 1.0.1-beta.0
git commit -m "feat: major change [alpha]"   # 1.0.0 → 1.1.0-beta.0
git commit -m "fix: continue beta [rc]"      # 1.0.0-beta.0 → 1.0.0-beta.1
git commit -m "feat: finalize [stable]"      # 1.0.0-beta.0 → 1.0.0
```

**Advanced Patterns:**
```bash
git commit -m "feat: patch beta [release:prepatch]"    # 1.0.0 → 1.0.1-beta.0
git commit -m "feat: minor beta [release:preminor]"    # 1.0.0 → 1.1.0-beta.0
git commit -m "feat: major beta [release:premajor]"    # 1.0.0 → 2.0.0-beta.0
git commit -m "feat: next beta [release:prerelease]"   # 1.0.0-beta.0 → 1.0.0-beta.1
```

**Manual Trigger:**
```yaml
# In your workflow file
workflow_dispatch:
  inputs:
    release-type:
      type: choice
      options: [patch, minor, major, prepatch, preminor, premajor, prerelease]
```

Then trigger manually selecting `prepatch` → 1.0.0 becomes 1.0.1-beta.0

### 5. Using PR Titles for Patterns

**Goal:** Control versioning through PR titles instead of commit messages

**Default Behavior:** The action automatically uses PR titles for pattern detection when triggered by `pull_request` events, and falls back to commit messages for `push` events.

**Usage (Zero Configuration):**
- PR title: `feat: Add user authentication [feature]` → Minor bump
- PR title: `fix: Critical security issue [hotfix]` → Patch bump  
- PR title: `refactor: Redesign API [breaking]` → Major bump
- PR title: `feat: New API endpoint [beta]` → 1.0.0 → 1.0.1-beta.0

**To Disable PR Pattern Detection:**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    enable-release: true
    release-use-pr-patterns: false  # Use commit messages only
```

**Custom PR Patterns:**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    enable-release: true
    release-version-patterns: |
      {
        "[deploy]": "patch",
        "[launch]": "minor"
      }
```

### 6. Skip Releases

**Goal:** Prevent releases for certain commits

**Default Skip Rules:**
- `[skip ci]` - Skips CI and release
- `[skip release]` - Skips only release  
- `chore(release):` - Skips release (used by action itself)

**Custom Skip Rules:**
```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    release-skip-rules: |
      [
        "[skip ci]",
        "[skip release]", 
        "[WIP]",
        "[draft]"
      ]
```

```bash
git commit -m "feat: work in progress [WIP]"     # No release
git commit -m "fix: not ready yet [draft]"      # No release
```

## Advanced Configuration

### Complete Example

```yaml
- uses: spanscale/node-github-action@v1.0.0
  with:
    # Basic settings
    enable-release: true
    release-create-changelog: true
    
    # Version control
    release-first-commit-bump: false
    release-version-patterns: |
      {
        "[release:v*]": "exact",
        "[hotfix]": "patch",
        "[breaking]": "major",
        "[beta]": "prepatch"
      }
    
    # Skip rules
    release-skip-rules: |
      [
        "[skip ci]",
        "[skip release]",
        "[WIP]",
        "chore(release):"
      ]
    
    # Git tagging
    release-create-tag: true
```

### Workflow Integration

**Prevent Release Loops:**
```yaml
on:
  push:
    branches: [main]

jobs:
  ci-cd:
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Important for changelog
          
      - uses: spanscale/node-github-action@v1.0.0
        with:
          enable-release: ${{ github.ref == 'refs/heads/main' }}  # Only on main
          # ... other config
```

## Troubleshooting

### Common Issues

**Q: My first commit bumped to 0.2.0 unexpectedly**
A: Set `release-first-commit-bump: false` or use non-feature commit messages

**Q: Releases are creating infinite loops**
A: Ensure your workflow only enables releases on specific branches and the default skip rules are preserved

**Q: I want version 2.0.0 but it became 2.0.0-beta.0**
A: Use exact version pattern: `[release:v2.0.0]` instead of prerelease types

**Q: No release was created despite having commits**
A: Check if your commit message matches skip rules, or if it's a non-bumping commit type (`docs:`, `test:`, etc.)

### Debug Information

The action provides detailed logging to help troubleshoot:
- Commit message analysis
- Pattern matching results
- Version bump decisions
- Skip rule evaluations

Check the action logs in GitHub Actions for detailed information about why a particular decision was made.

## Best Practices

1. **Consistent Commit Messages**: Train your team on conventional commits
2. **Clear Patterns**: Use obvious patterns like `[hotfix]` rather than cryptic ones
3. **Test First**: Try patterns on a test repository before production
4. **Document Team Patterns**: Share your custom patterns with the team
5. **Review Skip Rules**: Ensure skip rules don't accidentally prevent wanted releases

## Examples by Scenario

### Startup Project
```yaml
# Simple setup for new projects
release-first-commit-bump: false
release-version-patterns: '{"[v*]": "exact"}'
```

### Enterprise Project
```yaml
# Comprehensive setup with strict controls
release-first-commit-bump: false
release-version-patterns: |
  {
    "[release:v*]": "exact",
    "[hotfix]": "patch",
    "[security]": "patch",
    "[breaking]": "major",
    "[beta]": "prepatch"
  }
release-skip-rules: |
  [
    "[skip ci]", "[skip release]", "[WIP]", "[draft]",
    "[experimental]", "chore(release):"
  ]
```

### Open Source Library
```yaml
# Focus on semantic versioning with beta support
release-version-patterns: |
  {
    "[beta]": "prerelease",
    "[rc]": "prerelease",
    "[stable]": "patch"
  }
release-create-tag: true
```

This guide should help you choose the right configuration for your project's needs!

## Frequently Asked Questions (FAQ)

### General Questions

**Q: Do I need to understand conventional commits to use this action?**
A: Not necessarily! You can use custom patterns to override conventional commits entirely. For example, just use `[patch]`, `[minor]`, `[major]` in your commit messages.

**Q: What happens if I don't want any automatic versioning?**
A: Set `enable-release: false` or don't include the `enable-release` input at all. The action will only run CI scripts.

**Q: Can I use this action without pnpm?**
A: No, this action is specifically designed for pnpm projects. However, you can customize the `run-scripts` to use other package managers in your scripts.

**Q: Does the action use PR titles or commit messages for pattern detection?**
A: By default, it uses PR titles when triggered by `pull_request` events and commit messages for `push` events. You can disable PR pattern detection by setting `release-use-pr-patterns: false`.

### Version Bumping

**Q: Why did my version go from 1.0.0 to 1.0.0-beta.1 instead of 1.0.1?**
A: You likely used a prerelease bump type (`prepatch`, `preminor`, `premajor`, or `prerelease`) or your current version was already a prerelease. Check your custom patterns or manual trigger settings.

**Q: How do I go from 1.0.0-beta.1 back to 1.0.1?**
A: Use an exact version pattern: `git commit -m "release stable version [release:v1.0.1]"` or use the `[release:stable]` pattern with a patch-type commit.

**Q: Can I skip version bumping but still create a release?**
A: No, releases require version changes. Use skip rules like `[skip release]` to prevent releases entirely, or create releases manually outside the action.

**Q: What's the difference between `[release:patch]` and `[release:v1.0.1]`?**
A: `[release:patch]` increments the patch number (1.0.0 → 1.0.1), while `[release:v1.0.1]` sets the exact version to 1.0.1 regardless of the current version.

**Q: How do I bump from 1.0.0 to 1.0.1-beta.0?**
A: Use any of these patterns: `[beta]`, `[release:prepatch]`, or exact version `[release:v1.0.1-beta.0]`. The `[beta]` pattern is the simplest built-in option.

### Patterns and Rules

**Q: Can I use multiple patterns in one commit message?**
A: The action uses the first matching pattern it finds. Order matters in your `release-version-patterns` JSON object (though JSON technically doesn't guarantee order).

**Q: What if my commit message matches both a skip rule and a version pattern?**
A: Skip rules are checked first and take precedence. The release will be skipped.

**Q: Can I use regex in patterns?**
A: No, the patterns use simple string matching with `grep -q`. Use wildcards like `[release:v*]` for flexible matching.

**Q: How do I create a major version 2.0.0 from 1.5.3?**
A: Use exact version: `git commit -m "breaking: new API [release:v2.0.0]"` or conventional breaking change: `git commit -m "feat!: redesign API"`.

### Workflow Integration

**Q: Should I run this action on every branch or just main?**
A: Typically only on main/master for releases: `enable-release: ${{ github.ref == 'refs/heads/main' }}`. Run CI on all branches, releases only on main.

**Q: Why do I get "non-fast-forward" errors?**
A: This happens when multiple workflow runs try to push simultaneously. The action's skip rules should prevent this by skipping releases on the action's own commits.

**Q: Can I customize the changelog format?**
A: The action generates a standard changelog based on conventional commit categories. For custom formats, you'd need to disable `release-create-changelog` and handle it separately.

**Q: How do I publish to npmjs.org instead of GitHub Packages?**
A: Set `registry-url: 'https://registry.npmjs.org'` and provide `NPM_TOKEN` as a secret.

### Troubleshooting

**Q: My release was created but no tag was pushed**
A: Check if `release-create-tag: true` is set (it's the default). Also verify your `github-token` has push permissions.

**Q: The action says "No version bump needed" but I expected one**
A: Your commit message likely matches a skip rule or uses a non-bumping conventional commit type (`docs:`, `test:`, `chore:`, `ci:`, `style:`).

**Q: How do I test my configuration without creating real releases?**
A: Use a test repository, or temporarily set `release-publish-package: false` and `release-create-tag: false` to only test version bumping logic.

**Q: Can I see what version would be created without actually creating it?**
A: The action logs show the decision-making process. Look for "Determined bump type" and "New version" in the action logs.

### Best Practices

**Q: Should I squash commits before merging PRs?**
A: If you squash, make sure the squash commit message follows your versioning strategy. The action only sees the final commit message on the main branch.

**Q: How do I handle emergency hotfixes?**
A: Use `[hotfix]` pattern for immediate patch releases, or `[release:v1.2.4]` for exact version control. Consider a dedicated hotfix workflow if needed.

**Q: What's the recommended setup for a team new to semantic versioning?**
A: Start with default conventional commits, add `release-first-commit-bump: false`, and gradually introduce custom patterns as the team gets comfortable.

**Q: Should I create separate workflows for different release types?**
A: Usually not needed. One workflow with conditional `enable-release` based on branch and comprehensive pattern configuration covers most use cases.

### Advanced Scenarios

**Q: Can I create releases from feature branches?**
A: Yes, but set `enable-release` conditionally and be careful about version conflicts when merging back to main.

**Q: How do I handle breaking changes in patch releases?**
A: Don't! Breaking changes should increment the major version. If you must, use exact versioning with clear communication to users.

**Q: Can I integrate with external version management tools?**
A: The action is self-contained for simplicity. For complex version management, consider disabling the action's version bumping and using external tools.

**Q: What happens to version bumping with merge commits?**
A: The action examines the most recent commit, which would be the merge commit. Structure your merge commit messages according to your versioning patterns.