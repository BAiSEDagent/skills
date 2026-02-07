---
name: nextjs
description: Build production-ready Base dApps with Next.js, OnchainKit, and modern React patterns
trigger_phrases:
 - "build a next.js dapp"
 - "create base web app"
 - "setup onchainkit project"
 - "deploy to vercel"
 - "nextjs api routes for blockchain"
version: 1.0.0
---

# Next.js for Base dApp Development

## Guidelines

This skill teaches building production-grade decentralized applications on Base using Next.js 14+ with the App Router. Use this skill when:

- Creating full-stack Base dApps with server and client components
- Integrating Coinbase OnchainKit for wallet connection, transactions, and identity
- Building API routes for server-side blockchain interactions
- Deploying Base applications to Vercel or other hosting platforms
- Implementing modern React patterns with TypeScript
- Creating responsive UIs that interact with Base smart contracts

Next.js provides the ideal framework for Base dApps because it supports:
- Server-side rendering for better SEO and performance
- API routes for secure backend operations (contract reads, signature verification)
- Built-in TypeScript support and optimization
- Seamless Vercel deployment with environment management
- Hybrid rendering (SSR, SSG, ISR, Client Components)

This skill covers the complete workflow: project setup with create-next-app, OnchainKit integration for wallet functionality, API route handlers for blockchain backend logic, and production deployment with environment configuration.

Always use TypeScript, the App Router (not Pages Router), and follow Next.js 14+ conventions. Structure projects with clear separation between client components (wallet UI, transaction buttons) and server components (data fetching, contract reads).

For Base-specific features, leverage OnchainKit components which provide battle-tested UI for:
- Wallet connection and account management
- Transaction flows with smart transaction optimization
- Token swaps via Aerodrome and other DEXes
- Identity display with Basenames and avatars
- Funding via Coinbase Pay

Combine Next.js API routes with viem for server-side operations like:
- Reading contract state without exposing RPC endpoints
- Verifying signatures and processing webhooks
- Implementing gasless transaction relayers
- Caching blockchain data with ISR

Environment management is critical -- use separate .env.local for development and Vercel environment variables for production. Always configure Base mainnet (chain ID 8453) for production and Base Sepolia for testing.

## Examples

**Example 1: User says "create a nextjs dapp for token swaps"**
Agent creates a Next.js project with OnchainKit, sets up SwapDefault component, configures Aerodrome router integration, and creates API routes for fetching token prices.

**Example 2: User says "build api route to read nft metadata"**
Agent creates app/api/nft/[id]/route.ts using viem publicClient to read contract data server-side, implements caching with Next.js revalidation, returns JSON response.

**Example 3: User says "deploy my base dapp to vercel"**
Agent configures vercel.json, sets up environment variables (NEXT_PUBLIC_ONCHAINKIT_API_KEY, BASE_RPC_URL), optimizes build settings, and provides deployment commands.

## Resources

- Next.js Documentation: https://nextjs.org/docs
- Next.js App Router: https://nextjs.org/docs/app
- Coinbase OnchainKit: https://onchainkit.xyz
- OnchainKit GitHub: https://github.com/coinbase/onchainkit
- Base Network: https://docs.base.org
- Vercel Deployment: https://vercel.com/docs
- Viem Documentation: https://viem.sh
- Wagmi Documentation: https://wagmi.sh
- Base Mainnet RPC: https://mainnet.base.org

## Cross-References

- **typescript**: Essential for Next.js dApp development with type safety
- **viem-wagmi**: Core blockchain interaction libraries used in Next.js projects
- **git**: Version control for managing Next.js project code
- **onchainkit**: Detailed OnchainKit component usage and patterns
- **smart-contract-interaction**: Understanding contract calls from Next.js
- **base-deployment**: Broader Base deployment strategies beyond Vercel