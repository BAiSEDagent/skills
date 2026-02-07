---
name: hardhat
description: Hardhat development environment for Solidity smart contracts on Base
trigger_phrases:
  - "set up hardhat"
  - "hardhat testing"
  - "deploy with hardhat"
  - "hardhat configuration"
  - "hardhat plugins"
version: 1.0.0
---

# Hardhat Development Environment

## Guidelines

Use this skill when you need to set up, configure, test, or deploy Solidity smart contracts using Hardhat. Hardhat is a comprehensive development environment with built-in testing, deployment tools, and a rich plugin ecosystem.

Activate this skill when:
- Initializing a new Solidity project with TypeScript support
- Configuring network connections to Base (chain ID 8453)
- Writing and running comprehensive smart contract tests with fixtures
- Deploying contracts to Base mainnet or testnet using Ignition modules
- Setting up contract verification on Basescan
- Integrating plugins for gas reporting, coverage, or deployment management
- Creating custom Hardhat tasks for project automation
- Forking Base mainnet for local testing against live contracts
- Managing contract deployments with proxy patterns
- Running security analysis or gas optimization checks

Hardhat excels at:
- TypeScript-first development with full type safety
- Comprehensive testing with Mocha, Chai, and Hardhat matchers
- Network forking for testing against real deployed contracts
- Built-in console.log debugging in Solidity
- Flexible plugin architecture for extending functionality
- Ignition deployment system for reproducible deployments
- Integration with popular tools like Ethers.js and Viem

Key features:
- TypeScript configuration with hardhat.config.ts
- Built-in local Ethereum network (Hardhat Network)
- Stack traces and console.log for debugging
- Gas reporting and test coverage analysis
- Automatic contract verification on block explorers
- Deployment artifact management
- Task runner for custom automation

When configuring for Base:
- Use chain ID 8453 for mainnet, 84532 for Sepolia testnet
- Set RPC URL to https://mainnet.base.org for mainnet
- Configure etherscan.apiKey with Basescan API key
- Use customChains for Basescan verification URLs
- Set appropriate gas settings (Base uses EIP-1559)

Best practices:
- Use TypeScript for type safety in tests and scripts
- Organize tests with loadFixture for efficient setup
- Fork mainnet for integration testing against live protocols
- Use Ignition modules for repeatable deployments
- Enable gas reporter to optimize contract efficiency
- Run coverage reports to ensure comprehensive testing
- Verify contracts immediately after deployment
- Store deployment addresses in artifacts or JSON files
- Use environment variables for sensitive data (private keys, API keys)
- Create custom tasks for repetitive operations

## Examples

**Example 1: Initialize and configure Hardhat project**
User: "Set up a new Hardhat project for Base with TypeScript"
Agent: Creates project with `npx hardhat init`, configures hardhat.config.ts with Base network settings (RPC: https://mainhat.base.org, chainId: 8453), installs @nomicfoundation/hardhat-toolbox, sets up TypeScript strict mode, and creates example contract with tests.

**Example 2: Test contract with mainnet fork**
User: "Test my DeFi contract against live Aerodrome pools on Base"
Agent: Configures hardhat.config.ts with forking URL, writes test using loadFixture, impersonates whale accounts for token balance, interacts with Aerodrome Router (0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43), runs gas reporter, generates coverage report.

**Example 3: Deploy and verify contract**
User: "Deploy my contract to Base and verify on Basescan"
Agent: Creates Ignition deployment module, configures deployment parameters, deploys to Base mainnet, automatically verifies contract using hardhat-verify plugin with Basescan API, saves deployment addresses to JSON file.

## Resources

- Hardhat Documentation: https://hardhat.org/docs
- Hardhat TypeScript Setup: https://hardhat.org/guides/typescript
- Hardhat Testing Guide: https://hardhat.org/tutorial/testing-contracts
- Hardhat Ignition: https://hardhat.org/ignition/docs
- Hardhat Plugins: https://hardhat.org/plugins
- Base Network Documentation: https://docs.base.org
- Basescan: https://basescan.org
- Hardhat Toolbox: https://hardhat.org/hardhat-runner/plugins/nomicfoundation-hardhat-toolbox
- Hardhat Chai Matchers: https://hardhat.org/hardhat-chai-matchers

## Cross-References

- **solidity**: Core language for smart contracts compiled by Hardhat
- **foundry**: Alternative Solidity development framework with Rust-based tooling
- **typescript**: Language for Hardhat configuration, tests, and deployment scripts
- **git**: Version control for managing Hardhat projects and deployment history
- **viem**: TypeScript library for Ethereum interactions, alternative to Ethers.js in tests
- **etherscan**: Block explorer integration for contract verification