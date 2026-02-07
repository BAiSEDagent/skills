# GitHub API for Skill Automation

## Overview

This lesson covers programmatic GitHub repository management using the GitHub REST API. Autonomous AI agents use the API to create repositories, manage files, trigger workflows, and automate skill publishing. All examples use the Octokit SDK for type-safe API access and integrate with Base contract deployments.

## GitHub API Authentication

The GitHub API requires authentication via personal access tokens (PAT) or GitHub Apps. For autonomous agents, use fine-grained personal access tokens with minimal required permissions.

**Creating Fine-Grained Token**

```typescript
import { Octokit } from "@octokit/rest";

// Initialize Octokit with authentication
const octokit = new Octokit({
  auth: process.env.GITHUB_TOKEN,
  userAgent: "BAiSED-Agent/1.0.0",
  baseUrl: "https://api.github.com",
});

// Verify authentication
async function verifyAuth() {
  try {
    const { data: user } = await octokit.rest.users.getAuthenticated();
    console.log(`Authenticated as: ${user.login}`);
    return user;
  } catch (error) {
    console.error("Authentication failed:", error.message);
    throw error;
  }
}

// Required token permissions for skill management:
// - Contents: Read and write (for file operations)
// - Metadata: Read (for repository info)
// - Workflows: Read and write (for GitHub Actions)
// - Actions: Read and write (for workflow runs)
```

## Repository Management

Create and configure repositories for skill packages.

**Creating Repository**

```typescript
interface SkillRepoConfig {
  name: string;
  description: string;
  private: boolean;
  autoInit: boolean;
  gitignoreTemplate?: string;
  licenseTemplate?: string;
}

async function createSkillRepository(
  config: SkillRepoConfig
): Promise<string> {
  try {
    const { data: repo } = await octokit.rest.repos.createForAuthenticatedUser({
      name: config.name,
      description: config.description,
      private: config.private,
      auto_init: config.autoInit,
      gitignore_template: config.gitignoreTemplate,
      license_template: config.licenseTemplate,
    });

    console.log(`Repository created: ${repo.html_url}`);
    console.log(`Clone URL: ${repo.clone_url}`);

    // Configure repository settings
    await octokit.rest.repos.update({
      owner: repo.owner.login,
      repo: repo.name,
      has_issues: true,
      has_projects: false,
      has_wiki: false,
      allow_squash_merge: true,
      allow_merge_commit: true,
      allow_rebase_merge: true,
      delete_branch_on_merge: true,
    });

    return repo.full_name;
  } catch (error) {
    console.error("Failed to create repository:", error.message);
    throw error;
  }
}

// Example: Create repository for Aerodrome swap skill
const repoName = await createSkillRepository({
  name: "skill-swap-aerodrome",
  description: "BAiSED AI skill for token swaps via Aerodrome DEX on Base",
  private: false,
  autoInit: true,
  gitignoreTemplate: "Node",
  licenseTemplate: "mit",
});
```

**Managing Repository Settings**

```typescript
async function setupRepositoryForSkills(
  owner: string,
  repo: string
): Promise<void> {
  // Add topics for discoverability
  await octokit.rest.repos.replaceAllTopics({
    owner,
    repo,
    names: [
      "baised",
      "base",
      "coinbase",
      "ai-agent",
      "blockchain",
      "ethereum",
      "l2",
    ],
  });

  // Create branch protection for main
  await octokit.rest.repos.updateBranchProtection({
    owner,
    repo,
    branch: "main",
    required_status_checks: {
      strict: true,
      contexts: ["test", "build", "foundry-test"],
    },
    enforce_admins: true,
    required_pull_request_reviews: {
      dismiss_stale_reviews: true,
      require_code_owner_reviews: true,
      required_approving_review_count: 2,
    },
    restrictions: null,
    required_linear_history: true,
    allow_force_pushes: false,
    allow_deletions: false,
  });

  console.log(`Repository ${owner}/${repo} configured for skill development`);
}
```

## Contents API for File Management

The Contents API allows creating, reading, updating, and deleting files in repositories.

**Creating Skill Files**

```typescript
import { encode as base64Encode } from "base-64";

interface SkillFile {
  path: string;
  content: string;
  message: string;
}

async function createSkillFile(
  owner: string,
  repo: string,
  file: SkillFile
): Promise<void> {
  try {
    // GitHub API requires base64-encoded content
    const contentEncoded = base64Encode(file.content);

    const { data } = await octokit.rest.repos.createOrUpdateFileContents({
      owner,
      repo,
      path: file.path,
      message: file.message,
      content: contentEncoded,
      branch: "main",
    });

    console.log(`Created file: ${file.path}`);
    console.log(`Commit SHA: ${data.commit.sha}`);
  } catch (error) {
    console.error(`Failed to create file ${file.path}:`, error.message);
    throw error;
  }
}

// Example: Create SKILL.md for Moonwell lending skill
const skillMd = `---
name: lending-moonwell
description: Lending and borrowing USDC via Moonwell protocol on Base
trigger_phrases:
  - "supply USDC to Moonwell"
  - "borrow from Moonwell"
  - "check lending position"
version: 1.0.0
---

# Moonwell Lending on Base

## Guidelines

This skill enables AI agents to interact with Moonwell protocol on Base for USDC lending and borrowing. The Moonwell USDC market is deployed at 0xEdc817A28E8B93B03976FBd4a3dDBc9f7D176c22.

Use this skill to:
- Supply USDC as collateral and earn interest
- Borrow USDC against supplied collateral
- Monitor lending positions and health factors
- Integrate with reputation system for credit scoring

## Examples

Example 1: Supply USDC
Trigger: "Supply 1000 USDC to Moonwell"
Action: Approve USDC, call supply() on Moonwell market

Example 2: Borrow USDC
Trigger: "Borrow 500 USDC from Moonwell"
Action: Check collateral, call borrow() with amount

## Resources

- Moonwell Documentation: https://docs.moonwell.fi/
- Moonwell USDC Market: 0xEdc817A28E8B93B03976FBd4a3dDBc9f7D176c22
- Base Mainnet RPC: https://mainnet.base.org
`;

await createSkillFile("baised-ai", "skill-lending-moonwell", {
  path: "skills/lending-moonwell/SKILL.md",
  message: "feat(lending-moonwell): add skill documentation",
  content: skillMd,
});
```

**Batch File Creation**

```typescript
async function createSkillPackage(
  owner: string,
  repo: string,
  skillName: string,
  files: Map<string, string>
): Promise<void> {
  console.log(`Creating skill package: ${skillName}`);

  for (const [filename, content] of files.entries()) {
    const path = `skills/${skillName}/${filename}`;
    await createSkillFile(owner, repo, {
      path,
      content,
      message: `feat(${skillName}): add ${filename}`,
    });

    // Rate limiting: wait 100ms between requests
    await new Promise((resolve) => setTimeout(resolve, 100));
  }

  console.log(`Skill package ${skillName} created successfully`);
}

// Example: Create complete swap skill package
const swapSkillFiles = new Map<string, string>([
  [
    "SKILL.md",
    `---
name: swap-aerodrome
description: Token swaps via Aerodrome DEX on Base
trigger_phrases:
  - "swap tokens"
  - "trade on Aerodrome"
version: 1.0.0
---

# Aerodrome Swap Skill

## Guidelines
Execute token swaps using Aerodrome Router (0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43) on Base.
`,
  ],
  [
    "01-swapping.md",
    `# Token Swapping on Aerodrome

## Overview
This lesson covers single and multi-hop swaps via Aerodrome DEX.

## Contract Integration
Aerodrome Router: 0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43
Base Chain ID: 8453
`,
  ],
  [
    "02-routing.md",
    `# Multi-Hop Routing

## Overview
Optimal routing for complex token swaps.

## Implementation
Supports up to 4-hop routes for best pricing.
`,
  ],
]);

await createSkillPackage(
  "baised-ai",
  "skills-library",
  "swap-aerodrome",
  swapSkillFiles
);
```

**Reading and Updating Files**

```typescript
async function updateSkillVersion(
  owner: string,
  repo: string,
  skillName: string,
  newVersion: string
): Promise<void> {
  const path = `skills/${skillName}/SKILL.md`;

  // Get current file to obtain SHA
  const { data: currentFile } = await octokit.rest.repos.getContent({
    owner,
    repo,
    path,
  });

  if (Array.isArray(currentFile) || currentFile.type !== "file") {
    throw new Error(`${path} is not a file`);
  }

  // Decode current content
  const currentContent = Buffer.from(currentFile.content, "base64").toString(
    "utf-8"
  );

  // Update version in frontmatter
  const updatedContent = currentContent.replace(
    /version: [0-9]+\.[0-9]+\.[0-9]+/,
    `version: ${newVersion}`
  );

  // Update file with SHA for conflict detection
  const contentEncoded = base64Encode(updatedContent);
  await octokit.rest.repos.createOrUpdateFileContents({
    owner,
    repo,
    path,
    message: `chore(${skillName}): bump version to ${newVersion}`,
    content: contentEncoded,
    sha: currentFile.sha,
    branch: "main",
  });

  console.log(`Updated ${skillName} to version ${newVersion}`);
}
```

## GitHub Actions Integration

Trigger and monitor GitHub Actions workflows programmatically.

**Repository Dispatch Events**

```typescript
interface DispatchPayload {
  eventType: string;
  clientPayload: Record<string, any>;
}

async function triggerWorkflow(
  owner: string,
  repo: string,
  payload: DispatchPayload
): Promise<void> {
  try {
    await octokit.rest.repos.createDispatchEvent({
      owner,
      repo,
      event_type: payload.eventType,
      client_payload: payload.clientPayload,
    });

    console.log(`Triggered workflow: ${payload.eventType}`);
  } catch (error) {
    console.error("Failed to trigger workflow:", error.message);
    throw error;
  }
}

// Example: Trigger skill deployment to Base
await triggerWorkflow("baised-ai", "skill-swap-aerodrome", {
  eventType: "deploy-skill",
  clientPayload: {
    skill: "swap-aerodrome",
    version: "1.2.0",
    network: "base-mainnet",
    contracts: {
      router: "0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43",
      usdc: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    },
  },
});
```

**Monitoring Workflow Runs**

```typescript
async function waitForWorkflowCompletion(
  owner: string,
  repo: string,
  runId: number,
  timeoutMs: number = 600000 // 10 minutes
): Promise<string> {
  const startTime = Date.now();

  while (Date.now() - startTime < timeoutMs) {
    const { data: run } = await octokit.rest.actions.getWorkflowRun({
      owner,
      repo,
      run_id: runId,
    });

    console.log(`Workflow status: ${run.status} (${run.conclusion || "running"})`);

    if (run.status === "completed") {
      return run.conclusion;
    }

    // Wait 10 seconds before checking again
    await new Promise((resolve) => setTimeout(resolve, 10000));
  }

  throw new Error("Workflow timeout");
}

// Example: Trigger and wait for test workflow
async function runSkillTests(
  owner: string,
  repo: string,
  branch: string
): Promise<boolean> {
  // Trigger workflow
  await octokit.rest.actions.createWorkflowDispatch({
    owner,
    repo,
    workflow_id: "test.yml",
    ref: branch,
  });

  // Get most recent run
  const { data: runs } = await octokit.rest.actions.listWorkflowRuns({
    owner,
    repo,
    workflow_id: "test.yml",
    per_page: 1,
  });

  if (runs.workflow_runs.length === 0) {
    throw new Error("No workflow runs found");
  }

  const runId = runs.workflow_runs[0].id;
  const conclusion = await waitForWorkflowCompletion(owner, repo, runId);

  return conclusion === "success";
}
```

**Creating Workflow File**

```typescript
const testWorkflow = `name: Test Skill

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]
  repository_dispatch:
    types: [test-skill]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run TypeScript tests
        run: npm test
      
      - name: Setup Foundry
        uses: foundry-rs/foundry-toolchain@v1
      
      - name: Run Foundry tests against Base fork
        run: |
          forge test --fork-url https://mainnet.base.org --fork-block-number 10000000
        env:
          FOUNDRY_PROFILE: ci
      
      - name: Check gas usage
        run: forge snapshot --check
`;

await createSkillFile("baised-ai", "skill-swap-aerodrome", {
  path: ".github/workflows/test.yml",
  message: "ci: add test workflow",
  content: testWorkflow,
});
```

## Release Management

Automate release creation with changelogs and artifacts.

**Creating Releases**

```typescript
interface ReleaseConfig {
  tagName: string;
  name: string;
  body: string;
  draft: boolean;
  prerelease: boolean;
}

async function createRelease(
  owner: string,
  repo: string,
  config: ReleaseConfig
): Promise<string> {
  const { data: release } = await octokit.rest.repos.createRelease({
    owner,
    repo,
    tag_name: config.tagName,
    name: config.name,
    body: config.body,
    draft: config.draft,
    prerelease: config.prerelease,
    generate_release_notes: true,
  });

  console.log(`Release created: ${release.html_url}`);
  return release.html_url;
}

// Example: Create release for swap skill
const releaseUrl = await createRelease("baised-ai", "skill-swap-aerodrome", {
  tagName: "v1.2.0",
  name: "v1.2.0 - Multi-hop Routing",
  body: `## Features
- Add multi-hop swap routing through Aerodrome (0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43)
- Support up to 4 hops for optimal pricing
- Add slippage protection (default 0.5%)

## Improvements
- Optimize gas usage for single-hop swaps
- Add detailed swap simulation

## Base Integration
- Tested on Base mainnet (chain ID 8453)
- Compatible with USDC (0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913)
`,
  draft: false,
  prerelease: false,
});
```

## Common Patterns

**Rate Limiting**

```typescript
// GitHub API has rate limits: 5000 requests/hour for authenticated users
let requestCount = 0;
const RATE_LIMIT_MAX = 5000;
const RATE_LIMIT_WINDOW = 3600000; // 1 hour in ms

async function rateLimitedRequest<T>(
  apiCall: () => Promise<T>
): Promise<T> {
  requestCount++;

  if (requestCount >= RATE_LIMIT_MAX) {
    console.log("Rate limit approaching, waiting...");
    await new Promise((resolve) => setTimeout(resolve, RATE_LIMIT_WINDOW));
    requestCount = 0;
  }

  return apiCall();
}
```

**Error Handling**

```typescript
async function safeApiCall<T>(
  operation: string,
  apiCall: () => Promise<T>
): Promise<T | null> {
  try {
    return await apiCall();
  } catch (error: any) {
    if (error.status === 404) {
      console.log(`${operation}: Resource not found`);
      return null;
    } else if (error.status === 403) {
      console.error(`${operation}: Permission denied`);
      throw new Error("Insufficient permissions");
    } else if (error.status === 422) {
      console.error(`${operation}: Validation failed:`, error.message);
      throw error;
    } else {
      console.error(`${operation}: API error:`, error.message);
      throw error;
    }
  }
}
```

## Gotchas

**Base64 Encoding**: GitHub API requires file contents to be base64-encoded. Always encode before uploading and decode when reading.

**SHA for Updates**: When updating files, must provide current SHA to prevent conflicts. Always fetch current file first.

**File Size Limits**: Contents API has 100MB limit for files. For larger files, use Git Data API or Git LFS.

**Rate Limiting**: Authenticated requests limited to 5000/hour. Use conditional requests and caching when possible.

**Repository Dispatch Delay**: repository_dispatch events may take 1-2 minutes to trigger workflows. Don't expect immediate execution.

**Workflow Permissions**: Workflows need explicit permissions to write to repository. Set in workflow file or repository settings.

**Branch Protection**: Cannot push directly to protected branches via API. Must create PR and merge through review process.