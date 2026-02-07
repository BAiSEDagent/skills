---
name: git
description: Version control and GitHub workflows for BAiSED AI agent skill development and publishing
trigger_phrases:
  - "create a new git branch"
  - "commit my changes"
  - "push to GitHub"
  - "create a pull request"
  - "publish skill to registry"
  - "tag a release"
  - "setup CI/CD"
  - "configure git workflow"
version: 1.0.0
---

# Git Version Control for Skill Development

## Guidelines

This skill provides comprehensive version control workflows for developing, testing, and publishing BAiSED AI agent skills. Use this skill when:

- **Initializing new skill repositories**: Set up proper .gitignore, branch structure, and commit conventions for Solidity and TypeScript projects on Base
- **Managing skill development lifecycle**: Create feature branches, commit with conventional commit format, manage pull requests
- **Automating with GitHub Actions**: Configure CI/CD pipelines for contract testing with Foundry, TypeScript compilation, automated deployments to Base
- **Publishing skills to OpenClaw**: Tag releases, generate changelogs, distribute skills across multiple repositories
- **Collaborating on agent skills**: Review code, manage merge conflicts, synchronize with upstream changes
- **Integrating with Base contracts**: Version control for contracts interacting with Identity Registry (0x8004A169FB4a3325136EB29fA0ceB6D2e539a432), Reputation Registry (0x8004BAa17C55a88189AE136b182e5fdA19dE9b63)

The git skill follows trunk-based development adapted for blockchain projects. Main branch always contains production-ready code deployable to Base mainnet (chain ID 8453). Develop branch integrates tested features. Feature branches isolate individual skill improvements or new contract integrations.

Commit messages use conventional commits format: type(scope): description. Types include feat, fix, docs, test, refactor, chore. Scope identifies the affected skill or contract. This enables automated changelog generation and semantic versioning.

GitHub API integration allows programmatic repository management, automated file creation via Contents API, and triggering workflows through repository dispatch events. This supports autonomous skill publishing and cross-repository synchronization.

## Examples

**Example 1: Initialize Skill Repository**

Trigger: "Setup git repository for new swap skill"

Action:
```bash
# Initialize repository with proper structure
git init swap-skill
cd swap-skill

# Create .gitignore for Solidity/Node
cat > .gitignore << 'EOF'
node_modules/
.env
.env.local
out/
cache/
broadcast/
*.log
.DS_Store
lib/
artifacts/
typechain-types/
EOF

# Initialize branches
git checkout -b main
git commit --allow-empty -m "chore: initialize repository"
git checkout -b develop

# Create initial structure
mkdir -p skills/swap-aerodrome
touch skills/swap-aerodrome/SKILL.md
git add .
git commit -m "feat(swap-aerodrome): add skill structure"
```

**Example 2: Publish Skill Release**

Trigger: "Publish version 1.2.0 of the attestation skill"

Action:
```bash
# Ensure on main branch with latest changes
git checkout main
git pull origin main

# Tag release with semantic version
git tag -a v1.2.0 -m "feat: add EAS schema validation

- Add schema validation for attestations
- Integrate with EAS on Base (0x4200000000000000000000000000000000000021)
- Add revocation checks
- Update documentation"

# Push tag to trigger release workflow
git push origin v1.2.0

# GitHub Actions will automatically:
# 1. Run Foundry tests against Base fork
# 2. Build TypeScript packages
# 3. Generate changelog from conventional commits
# 4. Publish to OpenClaw registry
# 5. Create GitHub release with artifacts
```

**Example 3: Create Feature Branch for Contract Integration**

Trigger: "Create branch to add Moonwell lending integration"

Action:
```bash
# Create feature branch from develop
git checkout develop
git pull origin develop
git checkout -b feat/moonwell-lending

# Make changes to skill files
# Add Moonwell USDC Market contract (0xEdc817A28E8B93B03976FBd4a3dDBc9f7D176c22)

# Commit with conventional format
git add skills/lending-moonwell/
git commit -m "feat(lending-moonwell): add supply and borrow methods

Integrate with Moonwell USDC Market on Base:
- Add supply() function with USDC deposits
- Add borrow() function with collateral checks
- Add position monitoring via balanceOf
- Include example with reputation scoring"

# Push and create PR
git push -u origin feat/moonwell-lending
gh pr create --base develop --title "feat: Moonwell lending integration" \
  --body "Adds Moonwell protocol integration for USDC lending and borrowing on Base"
```

## Resources

- Git Documentation: https://git-scm.com/doc
- GitHub API Reference: https://docs.github.com/en/rest
- Conventional Commits: https://www.conventionalcommits.org/
- GitHub Actions: https://docs.github.com/en/actions
- Semantic Versioning: https://semver.org/
- Foundry CI/CD: https://book.getfoundry.sh/tutorials/best-practices#ci-cd

## Cross-References

- **openclaw-packaging**: Publishing skills to OpenClaw registry requires git tags and proper versioning
- **foundry**: Git workflows integrate with Foundry testing in CI/CD pipelines
- **viem-basics**: TypeScript contract code in skills uses viem and requires version control
- **reputation-system**: Reputation contracts deployed from git repositories with proper versioning