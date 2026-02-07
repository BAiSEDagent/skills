# Publishing Skills to OpenClaw Registry

## Overview

This lesson covers the complete skill publishing workflow for the OpenClaw registry. Publishing makes skills discoverable and usable by BAiSED AI agents on Base. The process includes git tagging, changelog generation, registry submission, and multi-repository distribution.

## OpenClaw Registry Overview

The OpenClaw registry is a decentralized skill package manager for AI agents operating on Base (chain ID 8453). Skills are published as git repositories with semantic versioning. The registry indexes skills by name, version, and capabilities.

**Registry Structure**

```typescript
interface SkillPackage {
  name: string; // Must match directory name
  version: string; // Semantic versioning (1.0.0)
  description: string;
  author: string;
  repository: string; // Git repository URL
  triggerPhrases: string[];
  baseContracts: string[]; // Deployed contract addresses
  dependencies: Record<string, string>; // Other skills required
  files: string[]; // Skill files to include
}

// Example skill package metadata
const swapSkillPackage: SkillPackage = {
  name: "swap-aerodrome",
  version: "1.2.0",
  description: "Token swaps via Aerodrome DEX on Base",
  author: "BAiSED AI",
  repository: "https://github.com/baised-ai/skill-swap-aerodrome",
  triggerPhrases: ["swap tokens", "trade on Aerodrome", "exchange via DEX"],
  baseContracts: [
    "0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43", // Aerodrome Router
    "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913", // USDC
  ],
  dependencies: {
    "viem-basics": "^1.0.0",
    "wallet-management": "^1.1.0",
  },
  files: ["SKILL.md", "01-swapping.md", "02-routing.md"],
};
```

## Tagging Releases

Semantic versioning tags mark publishable releases. Tags trigger automated publishing workflows.

**Creating Version Tags**

```bash
# Ensure on main branch with all changes committed
git checkout main
git pull origin main

# Create annotated tag with detailed message
git tag -a v1.2.0 -m "feat: Multi-hop routing for Aerodrome swaps

New Features:
- Multi-hop swap routing through Aerodrome Router (0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43)
- Support for up to 4-hop routes
- Automatic route optimization for best prices
- Slippage protection with configurable tolerance

Improvements:
- Reduce gas costs for single-hop swaps by 15%
- Add swap simulation before execution
- Improve error messages for failed swaps

Base Integration:
- Tested on Base mainnet (chain ID 8453)
- Compatible with all major Base tokens
- Integrated with Base USDC (0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913)

Breaking Changes:
- None

Migration:
- No migration required

Documentation:
- Updated SKILL.md with multi-hop examples
- Added routing algorithm explanation
- Included gas optimization tips"

# Push tag to trigger publishing workflow
git push origin v1.2.0

# Push all tags
git push origin --tags
```

**Semantic Versioning Rules**

```bash
# MAJOR version (x.0.0): Breaking changes
# - Change function signatures
# - Remove features
# - Change contract interfaces
git tag -a v2.0.0 -m "feat!: Redesign swap interface

BREAKING CHANGE: swapTokens() now requires RouteParams struct instead of individual parameters."

# MINOR version (1.x.0): New features, backwards compatible
# - Add new functions
# - Add new trigger phrases
# - Extend functionality
git tag -a v1.3.0 -m "feat(swap): Add limit order support

Adds limit order functionality for deferred swaps."

# PATCH version (1.2.x): Bug fixes, no new features
# - Fix bugs
# - Update documentation
# - Performance improvements
git tag -a v1.2.1 -m "fix(swap): Handle zero-liquidity pools gracefully

Prevents revert when pool has no liquidity."
```

**Programmatic Tagging**

```typescript
import { simpleGit, SimpleGit } from "simple-git";

const git: SimpleGit = simpleGit();

async function createReleaseTag(
  version: string,
  message: string
): Promise<void> {
  try {
    // Verify on main branch
    const status = await git.status();
    if (status.current !== "main") {
      throw new Error("Must be on main branch to create release tag");
    }

    // Verify no uncommitted changes
    if (!status.isClean()) {
      throw new Error("Working directory must be clean");
    }

    // Pull latest changes
    await git.pull("origin", "main");

    // Create annotated tag
    await git.addAnnotatedTag(`v${version}`, message);

    // Push tag
    await git.pushTags("origin");

    console.log(`Created and pushed tag v${version}`);
  } catch (error) {
    console.error("Failed to create release tag:", error.message);
    throw error;
  }
}

// Example: Create tag for reputation system update
await createReleaseTag(
  "1.4.0",
  `feat(reputation): Add on-chain attestation integration

Integrates with EAS on Base (0x4200000000000000000000000000000000000021) for on-chain reputation attestations.

Features:
- Create attestations with reputation scores
- Query existing attestations
- Revoke attestations
- Schema validation

Contracts:
- EAS: 0x4200000000000000000000000000000000000021
- Schema Registry: 0x4200000000000000000000000000000000000020
- Reputation Registry: 0x8004BAa17C55a88189AE136b182e5fdA19dE9b63`
);
```

## Changelog Generation

Generate changelogs automatically from conventional commits.

**Using Conventional Changelog**

```bash
# Install conventional-changelog-cli
npm install --save-dev conventional-changelog-cli

# Generate changelog for current version
npx conventional-changelog -p angular -i CHANGELOG.md -s

# Generate full changelog from all commits
npx conventional-changelog -p angular -i CHANGELOG.md -s -r 0

# Preview without writing to file
npx conventional-changelog -p angular
```

**Example CHANGELOG.md**

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.2.0] - 2024-01-15

### Features

- **swap-aerodrome**: Add multi-hop routing through Aerodrome Router (0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43)
- **swap-aerodrome**: Support up to 4-hop routes for optimal pricing
- **swap-aerodrome**: Add slippage protection with configurable tolerance (default 0.5%)

### Improvements

- **swap-aerodrome**: Reduce gas costs for single-hop swaps by 15%
- **swap-aerodrome**: Add swap simulation before execution
- **swap-aerodrome**: Improve error messages for failed swaps

### Base Integration

- Tested on Base mainnet (chain ID 8453)
- Compatible with USDC (0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913)
- Integration with Aerodrome DEX verified

### Documentation

- Update SKILL.md with multi-hop examples
- Add routing algorithm explanation
- Include gas optimization tips

## [1.1.0] - 2024-01-01

### Features

- **swap-aerodrome**: Add price impact calculation
- **swap-aerodrome**: Add deadline parameter for time-sensitive swaps

### Bug Fixes

- **swap-aerodrome**: Fix decimal handling for tokens with non-18 decimals
- **swap-aerodrome**: Handle zero-liquidity pools gracefully

## [1.0.0] - 2023-12-15

### Features

- Initial release of Aerodrome swap skill
- Single-hop token swaps
- Integration with Aerodrome Router on Base
- USDC swap examples
```

**Automated Changelog Generation**

```typescript
import { exec } from "child_process";
import { promisify } from "util";
import * as fs from "fs/promises";

const execAsync = promisify(exec);

async function generateChangelog(
  fromTag?: string,
  toTag: string = "HEAD"
): Promise<string> {
  try {
    // Generate changelog using conventional-changelog
    const cmd = fromTag
      ? `npx conventional-changelog -p angular --tag-prefix v --from ${fromTag} --to ${toTag}`
      : `npx conventional-changelog -p angular --tag-prefix v`;

    const { stdout } = await execAsync(cmd);
    return stdout;
  } catch (error) {
    console.error("Failed to generate changelog:", error.message);
    throw error;
  }
}

async function updateChangelogFile(version: string): Promise<void> {
  const changelogPath = "CHANGELOG.md";

  // Generate new changelog entries
  const newEntries = await generateChangelog(`v${getPreviousVersion(version)}`);

  // Read existing changelog
  let existingChangelog = "";
  try {
    existingChangelog = await fs.readFile(changelogPath, "utf-8");
  } catch (error) {
    // File doesn't exist, create header
    existingChangelog = `# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

`;
  }

  // Insert new entries after header
  const headerEnd = existingChangelog.indexOf("## ");
  const updatedChangelog =
    existingChangelog.slice(0, headerEnd) +
    newEntries +
    "\n" +
    existingChangelog.slice(headerEnd);

  // Write updated changelog
  await fs.writeFile(changelogPath, updatedChangelog);
  console.log(`Updated ${changelogPath} for version ${version}`);
}

function getPreviousVersion(currentVersion: string): string {
  const [major, minor, patch] = currentVersion.split(".").map(Number);
  if (patch > 0) return `${major}.${minor}.${patch - 1}`;
  if (minor > 0) return `${major}.${minor - 1}.0`;
  if (major > 0) return `${major - 1}.0.0`;
  return "0.0.0";
}
```

## Publishing to OpenClaw Registry

Submit skills to the OpenClaw registry for agent discovery.

**Registry Submission**

```typescript
import { createPublicClient, createWalletClient, http, parseAbi } from "viem";
import { base } from "viem/chains";
import { privateKeyToAccount } from "viem/accounts";

// OpenClaw Registry contract (example address)
const OPENCLAW_REGISTRY = "0x1234567890123456789012345678901234567890";

const registryAbi = parseAbi([
  "function publishSkill(string name, string version, string repositoryUrl, bytes32 contentHash) external",
  "function updateSkill(string name, string version, string repositoryUrl, bytes32 contentHash) external",
  "function getSkill(string name, string version) external view returns (tuple(string repository, bytes32 contentHash, uint256 publishedAt, address publisher))",
  "event SkillPublished(string indexed name, string indexed version, address indexed publisher)",
]);

const account = privateKeyToAccount(
  process.env.PUBLISHER_PRIVATE_KEY as `0x${string}`
);

const walletClient = createWalletClient({
  account,
  chain: base,
  transport: http("https://mainnet.base.org"),
});

const publicClient = createPublicClient({
  chain: base,
  transport: http("https://mainnet.base.org"),
});

async function publishSkillToRegistry(
  name: string,
  version: string,
  repositoryUrl: string,
  contentHash: string
): Promise<string> {
  try {
    // Submit skill to registry contract
    const hash = await walletClient.writeContract({
      address: OPENCLAW_REGISTRY,
      abi: registryAbi,
      functionName: "publishSkill",
      args: [name, version, repositoryUrl, contentHash as `0x${string}`],
    });

    console.log(`Publishing skill ${name}@${version}...`);
    console.log(`Transaction: ${hash}`);

    // Wait for confirmation
    const receipt = await publicClient.waitForTransactionReceipt({ hash });

    console.log(
      `Skill published in block ${receipt.blockNumber} on Base (chain ID 8453)`
    );
    return hash;
  } catch (error) {
    console.error("Failed to publish skill:", error.message);
    throw error;
  }
}

// Example: Publish swap skill
import { createHash } from "crypto";

function calculateContentHash(content: string): string {
  return "0x" + createHash("sha256").update(content).digest("hex");
}

const skillContent = await fs.readFile(
  "skills/swap-aerodrome/SKILL.md",
  "utf-8"
);
const contentHash = calculateContentHash(skillContent);

await publishSkillToRegistry(
  "swap-aerodrome",
  "1.2.0",
  "https://github.com/baised-ai/skill-swap-aerodrome",
  contentHash
);
```

**Publishing Workflow**

```yaml
# .github/workflows/publish.yml
name: Publish Skill to OpenClaw

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Extract version from tag
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
      
      - name: Generate changelog
        run: |
          npx conventional-changelog -p angular -i CHANGELOG.md -s
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add CHANGELOG.md
          git commit -m "chore: update changelog for v${{ steps.version.outputs.version }}"
          git push origin HEAD:main
      
      - name: Run tests
        run: npm test
      
      - name: Build packages
        run: npm run build
      
      - name: Publish to OpenClaw registry
        env:
          PUBLISHER_PRIVATE_KEY: ${{ secrets.PUBLISHER_PRIVATE_KEY }}
        run: |
          node scripts/publish-to-registry.js \
            --name swap-aerodrome \
            --version ${{ steps.version.outputs.version }} \
            --repository ${{ github.repositoryUrl }}
      
      - name: Create GitHub release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v${{ steps.version.outputs.version }}
          release_name: v${{ steps.version.outputs.version }}
          body_path: CHANGELOG.md
          draft: false
          prerelease: false
```

## Multi-Repository Skill Distribution

Distribute skills across multiple repositories for modular development.

**Skill Dependencies**

```json
// package.json for swap-aerodrome skill
{
  "name": "@baised-ai/skill-swap-aerodrome",
  "version": "1.2.0",
  "description": "Aerodrome DEX swap skill for BAiSED AI agents on Base",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "publish-skill": "node scripts/publish-to-registry.js"
  },
  "dependencies": {
    "viem": "^2.0.0",
    "@baised-ai/skill-viem-basics": "^1.0.0",
    "@baised-ai/skill-wallet-management": "^1.1.0"
  },
  "peerDependencies": {
    "@baised-ai/agent-core": "^2.0.0"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/baised-ai/skill-swap-aerodrome.git"
  },
  "keywords": [
    "baised",
    "base",
    "aerodrome",
    "dex",
    "swap",
    "ai-agent"
  ]
}
```

**Git Submodules for Monorepos**

```bash
# Add skill as submodule to main skills repository
git submodule add https://github.com/baised-ai/skill-swap-aerodrome.git skills/swap-aerodrome

# Initialize and update all submodules
git submodule init
git submodule update --recursive --remote

# Commit submodule addition
git add .gitmodules skills/swap-aerodrome
git commit -m "feat: add swap-aerodrome skill as submodule"

# Update specific submodule to latest version
cd skills/swap-aerodrome
git checkout v1.2.0
cd ../..
git add skills/swap-aerodrome
git commit -m "chore: update swap-aerodrome to v1.2.0"
```

**Skill Registry Index**

```typescript
// skills/index.ts - Central registry of all skills
export const SKILL_REGISTRY = [
  {
    name: "swap-aerodrome",
    version: "1.2.0",
    repository: "https://github.com/baised-ai/skill-swap-aerodrome",
    package: "@baised-ai/skill-swap-aerodrome",
    baseContracts: ["0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43"],
  },
  {
    name: "lending-moonwell",
    version: "1.0.0",
    repository: "https://github.com/baised-ai/skill-lending-moonwell",
    package: "@baised-ai/skill-lending-moonwell",
    baseContracts: ["0xEdc817A28E8B93B03976FBd4a3dDBc9f7D176c22"],
  },
  {
    name: "reputation-system",
    version: "1.4.0",
    repository: "https://github.com/baised-ai/skill-reputation-system",
    package: "@baised-ai/skill-reputation-system",
    baseContracts: [
      "0x8004A169FB4a3325136EB29fA0ceB6D2e539a432",
      "0x8004BAa17C55a88189AE136b182e5fdA19dE9b63",
    ],
  },
];

// Load skill dynamically
export async function loadSkill(name: string): Promise<any> {
  const skill = SKILL_REGISTRY.find((s) => s.name === name);
  if (!skill) {
    throw new Error(`Skill not found: ${name}`);
  }

  // Dynamic import of skill package
  return import(skill.package);
}
```

## Common Patterns

**Pre-release Versions**

```bash
# Alpha release (early testing)
git tag -a v1.3.0-alpha.1 -m "feat: Add limit orders (alpha)"

# Beta release (feature complete, testing)
git tag -a v1.3.0-beta.1 -m "feat: Add limit orders (beta)"

# Release candidate (final testing)
git tag -a v1.3.0-rc.1 -m "feat: Add limit orders (release candidate)"

# Stable release
git tag -a v1.3.0 -m "feat: Add limit orders"
```

**Hotfix Releases**

```bash
# Create hotfix branch from tag
git checkout -b hotfix/1.2.1 v1.2.0

# Fix critical bug
git commit -m "fix: prevent overflow in swap calculation"

# Merge back to main
git checkout main
git merge --no-ff hotfix/1.2.1

# Tag hotfix
git tag -a v1.2.1 -m "fix: prevent overflow in swap calculation"
git push origin main v1.2.1

# Merge to develop
git checkout develop
git merge --no-ff hotfix/1.2.1
git push origin develop
```

## Gotchas

**Tag Annotation**: Always use annotated tags (`-a`) for releases, not lightweight tags. Annotated tags include author, date, and message.

**Version Prefix**: Consistently use `v` prefix for version tags (v1.2.0, not 1.2.0). This is convention in most projects.

**Changelog Timing**: Generate and commit changelog before creating release tag, so changelog is included in release.

**Breaking Changes**: Always mark breaking changes with `!` in commit type or `BREAKING CHANGE:` in footer. This triggers major version bump.

**Registry Content Hash**: Content hash must match actual skill content. Regenerate hash if files change after initial calculation.

**Submodule Updates**: Submodules don't auto-update. Must explicitly run `git submodule update --remote` and commit changes.

**Multiple Registries**: Skills may be published to multiple registries (OpenClaw, npm, PyPI). Ensure version synchronization across all registries.