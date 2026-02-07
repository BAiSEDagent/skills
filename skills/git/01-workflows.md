# Git Workflows for Skill Development

## Overview

This lesson covers branch strategies, commit conventions, and pull request workflows optimized for BAiSED AI agent skill development on Base. Git workflows ensure code quality, enable collaboration, and support automated publishing to the OpenClaw registry. All workflows integrate with Base mainnet (chain ID 8453) deployment pipelines and smart contract testing via Foundry.

## Branch Strategy

The BAiSED skills library uses a modified trunk-based development model with three primary branch types:

**Main Branch**

The main branch contains production-ready code deployed to Base mainnet. All contracts in main have been tested against Base fork and audited. Merges to main trigger automatic deployment pipelines and registry publishing.

```bash
# Main branch protection rules
# - Require pull request reviews (2 approvals)
# - Require status checks (Foundry tests, TypeScript build)
# - Require branches to be up to date
# - No direct pushes allowed

# Update main from develop (release)
git checkout main
git merge --no-ff develop -m "chore: release v1.3.0"
git tag v1.3.0
git push origin main --tags
```

**Develop Branch**

The develop branch integrates completed features and fixes. Code in develop has passed all tests but may not be production-ready. This branch deploys to Base Sepolia testnet for integration testing.

```bash
# Create develop from main (first time)
git checkout -b develop main
git push -u origin develop

# Merge feature into develop
git checkout develop
git pull origin develop
git merge --no-ff feat/eas-integration -m "feat: merge EAS attestation skill"
git push origin develop
```

**Feature Branches**

Feature branches implement individual skills, contract integrations, or bug fixes. Branch names follow the pattern: type/description (feat/swap-aerodrome, fix/reputation-overflow, docs/viem-examples).

```bash
# Create feature branch
git checkout develop
git pull origin develop
git checkout -b feat/cctp-bridge

# Work on feature with atomic commits
touch skills/bridge-cctp/SKILL.md
git add skills/bridge-cctp/SKILL.md
git commit -m "feat(bridge-cctp): add skill documentation"

touch skills/bridge-cctp/01-usdc-bridging.md
git add skills/bridge-cctp/01-usdc-bridging.md
git commit -m "feat(bridge-cctp): add USDC bridging lesson

Integrates with CCTP TokenMessenger on Base (0x1682Ae6375C4E4A97e4B583BC394c861A46D8962) for cross-chain USDC transfers."

# Push feature branch
git push -u origin feat/cctp-bridge
```

## Commit Conventions

All commits follow the Conventional Commits specification. This enables automated changelog generation, semantic versioning, and integration with CI/CD pipelines.

**Commit Format**

```
type(scope): subject

body (optional)

footer (optional)
```

**Types:**
- **feat**: New skill or feature (triggers minor version bump)
- **fix**: Bug fix (triggers patch version bump)
- **docs**: Documentation changes only
- **test**: Adding or updating tests
- **refactor**: Code changes that neither fix bugs nor add features
- **chore**: Maintenance tasks, dependency updates
- **perf**: Performance improvements
- **breaking**: Breaking changes (triggers major version bump, use BREAKING CHANGE: in footer)

**Scope Examples:**
- skill name: swap-aerodrome, lending-moonwell, reputation-system
- contract name: IdentityRegistry, ReputationScorer
- component: api, cli, sdk

**Commit Examples**

```bash
# Feature commit
git commit -m "feat(swap-aerodrome): add multi-hop routing

Implements multi-hop swap routing through Aerodrome Router (0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43). Supports up to 4 hops for optimal pricing.

- Add route calculation function
- Add slippage protection (default 0.5%)
- Add deadline parameter (default 20 minutes)
- Include example swapping USDC to ETH via AERO"

# Fix commit
git commit -m "fix(reputation-system): prevent integer overflow in score calculation

Adds SafeMath checks to reputation score calculation in ReputationScorer contract deployed at 0x8004BAa17C55a88189AE136b182e5fdA19dE9b63."

# Documentation commit
git commit -m "docs(viem-basics): add contract read examples

Adds examples for reading from Identity Registry (0x8004A169FB4a3325136EB29fA0ceB6D2e539a432) using viem publicClient."

# Breaking change commit
git commit -m "feat(identity)!: change registry interface

BREAKING CHANGE: getIdentity() now returns struct instead of tuple. Update all calls to use named fields.

Old: const [addr, basename] = await getIdentity(id)
New: const {owner, basename} = await getIdentity(id)"
```

## Pull Request Workflow

Pull requests (PRs) are the primary method for code review and integration. All PRs must pass automated checks before merging.

**Creating Pull Requests**

```bash
# Using GitHub CLI
gh pr create \
  --base develop \
  --title "feat(lending-moonwell): add Moonwell USDC market integration" \
  --body "## Description
Adds lending and borrowing functionality for Moonwell USDC market on Base.

## Changes
- Integrated with Moonwell USDC Market (0xEdc817A28E8B93B03976FBd4a3dDBc9f7D176c22)
- Added supply() and borrow() functions
- Added position monitoring
- Included reputation score integration

## Testing
- [x] Unit tests pass
- [x] Integration tests against Base fork
- [x] Manual testing on Base Sepolia

## Checklist
- [x] Follows conventional commit format
- [x] Documentation updated
- [x] Tests added/updated
- [x] No breaking changes"
```

**PR Template (.github/pull_request_template.md)**

```markdown
## Description

<!-- Describe your changes in detail -->

## Type of Change

- [ ] feat: New skill or feature
- [ ] fix: Bug fix
- [ ] docs: Documentation update
- [ ] refactor: Code refactoring
- [ ] test: Adding or updating tests
- [ ] chore: Maintenance tasks

## Contracts Affected

<!-- List any Base contracts this PR interacts with -->
- [ ] Identity Registry (0x8004A169FB4a3325136EB29fA0ceB6D2e539a432)
- [ ] Reputation Registry (0x8004BAa17C55a88189AE136b182e5fdA19dE9b63)
- [ ] EAS (0x4200000000000000000000000000000000000021)
- [ ] Other: 

## Testing

- [ ] Unit tests pass locally
- [ ] Integration tests pass against Base fork
- [ ] Tested on Base Sepolia testnet
- [ ] Gas usage analyzed and optimized

## Checklist

- [ ] Commits follow conventional commit format
- [ ] Code follows project style guidelines
- [ ] Documentation updated (SKILL.md, lesson files)
- [ ] Tests added for new functionality
- [ ] All tests passing
- [ ] No breaking changes (or marked with !)
```

## Rebasing vs Merging

**Use Rebase For:**

Feature branches to keep linear history and incorporate upstream changes.

```bash
# Update feature branch with latest develop
git checkout feat/eas-integration
git fetch origin
git rebase origin/develop

# If conflicts occur
# 1. Resolve conflicts in files
# 2. Stage resolved files
git add resolved-file.ts
# 3. Continue rebase
git rebase --continue

# Force push rebased branch (only for feature branches)
git push --force-with-lease origin feat/eas-integration
```

**Use Merge For:**

Integrating feature branches into develop or develop into main. Preserves branch history and provides merge commit for tracking.

```bash
# Merge feature into develop (with no fast-forward)
git checkout develop
git merge --no-ff feat/eas-integration -m "feat: integrate EAS attestation skill

Merges EAS attestation functionality:
- Schema creation and validation
- Attestation creation with reputation data
- Revocation handling
- Integration with Base EAS contract (0x4200000000000000000000000000000000000021)"

git push origin develop
```

## .gitignore for Solidity and Node Projects

Proper .gitignore prevents committing build artifacts, dependencies, and sensitive data.

```gitignore
# Dependencies
node_modules/
package-lock.json
yarn.lock

# Environment variables
.env
.env.local
.env.development
.env.test
.env.production
*.env

# Foundry/Solidity
out/
cache/
broadcast/
lib/
*.txt
!foundry.toml

# Hardhat
artifacts/
cache/
typechain-types/

# Testing
coverage/
coverage.json
.coverage_cache/

# Logs
*.log
logs/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Build outputs
dist/
build/
*.js.map
*.d.ts.map

# Temporary files
*.tmp
*.temp
.cache/
```

**Project-Specific .gitignore**

```bash
# For skills repository
cat >> .gitignore << 'EOF'
# Skill-specific build outputs
skills/*/dist/
skills/*/node_modules/

# Generated documentation
docs/generated/

# Local testing
local-data/
test-wallets.json
EOF
```

## Common Patterns

**Interactive Rebase for Commit Cleanup**

```bash
# Clean up commits before creating PR
git rebase -i HEAD~5

# In editor, change 'pick' to:
# - 'squash' (or 's') to combine with previous commit
# - 'reword' (or 'r') to change commit message
# - 'edit' (or 'e') to modify commit
# - 'drop' (or 'd') to remove commit

# Example:
pick abc1234 feat(swap): add aerodrome integration
squash def5678 fix typo
squash ghi9012 add tests
reword jkl3456 docs: update examples
```

**Cherry-Pick Commits**

```bash
# Apply specific commit from another branch
git checkout develop
git cherry-pick abc1234

# Cherry-pick without committing (for modifications)
git cherry-pick -n abc1234
# Make changes
git add .
git commit -m "feat(swap): cherry-pick aerodrome integration with modifications"
```

**Stash Changes**

```bash
# Save work in progress
git stash push -m "WIP: reputation calculation optimization"

# List stashes
git stash list

# Apply most recent stash
git stash pop

# Apply specific stash
git stash apply stash@{2}
```

## Gotchas

**Never Rebase Public Branches**: Only rebase feature branches that haven't been merged. Never rebase main or develop branches, as this rewrites history and breaks collaboration.

**Force Push Safety**: Always use `git push --force-with-lease` instead of `git push --force`. This prevents accidentally overwriting others' work.

**Commit Message Length**: Keep subject line under 72 characters. Use body for detailed explanations. GitHub truncates long subject lines in UI.

**Binary Files**: Never commit compiled contracts (.json artifacts), node_modules, or environment files. These bloat repository and may contain secrets.

**Merge Conflicts in Package Files**: When package.json or package-lock.json conflict, often easiest to accept incoming changes and reinstall dependencies rather than manually merging.

**Detached HEAD State**: If `git status` shows "HEAD detached at...", you're not on a branch. Create a branch to save work: `git checkout -b rescue-branch`.

**Sensitive Data in History**: If secrets committed, don't just delete in new commit. Use `git filter-branch` or BFG Repo-Cleaner to remove from history, then force push and rotate secrets.