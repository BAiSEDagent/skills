# Next.js Project Setup for Base dApps

## Overview

This lesson covers setting up a production-ready Next.js project for Base blockchain development. You will learn to initialize a Next.js 14+ project with TypeScript and App Router, install essential Base development dependencies (wagmi, viem, OnchainKit), structure your project directories, and configure environment variables for Base mainnet and testnet.

## Creating a Next.js Project

Use create-next-app with TypeScript and App Router:

```bash
npx create-next-app@latest my-base-dapp
```

When prompted, select:
- TypeScript: Yes
- ESLint: Yes
- Tailwind CSS: Yes
- src/ directory: No (optional, but recommended for clarity)
- App Router: Yes
- Import alias: Yes (@/* recommended)

This creates a project structure:

```
my-base-dapp/
|-- app/
| |-- layout.tsx
| |-- page.tsx
| |-- globals.css
|-- public/
|-- package.json
|-- tsconfig.json
|-- next.config.js
|-- tailwind.config.ts
```

## Installing Base Development Dependencies

Install wagmi, viem, and OnchainKit:

```bash
cd my-base-dapp
npm install wagmi viem@2.x @coinbase/onchainkit
npm install @tanstack/react-query
```

These packages provide:
- **wagmi**: React hooks for Ethereum/Base interaction
- **viem**: TypeScript Ethereum library (replaces ethers.js)
- **@coinbase/onchainkit**: Coinbase UI components for Base
- **@tanstack/react-query**: Required peer dependency for wagmi

Verify installation in package.json:

```json
{
 "dependencies": {
 "@coinbase/onchainkit": "^0.28.0",
 "@tanstack/react-query": "^5.28.0",
 "next": "14.2.0",
 "react": "^18.3.0",
 "react-dom": "^18.3.0",
 "viem": "^2.9.0",
 "wagmi": "^2.5.0"
 }
}
```

## Project Structure for Base dApps

Organize your Next.js Base dApp:

```
my-base-dapp/
|-- app/
| |-- layout.tsx # Root layout with providers
| |-- page.tsx # Home page
| |-- api/ # API route handlers
| | |-- contract/
| | | |-- read/
| | | | |-- route.ts
| |-- components/ # React components
| | |-- wallet-connect.tsx
| | |-- transaction-button.tsx
| |-- providers.tsx # Wagmi/OnchainKit providers
|-- lib/
| |-- contracts.ts # Contract ABIs and addresses
| |-- config.ts # Wagmi config
| |-- utils.ts # Helper functions
|-- public/
| |-- images/
|-- .env.local # Local environment variables
|-- .env.example # Example environment file
```

Create lib/config.ts for wagmi configuration:

```typescript
import { http, createConfig } from "wagmi";
import { base, baseSepolia } from "wagmi/chains";
import { coinbaseWallet, injected, walletConnect } from "wagmi/connectors";

const projectId = process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID || "";

export const config = createConfig({
 chains: [base, baseSepolia],
 connectors: [
 injected(),
 coinbaseWallet({
 appName: "My Base dApp",
 preference: "smartWalletOnly",
 }),
 walletConnect({ projectId }),
 ],
 transports: {
 [base.id]: http(process.env.NEXT_PUBLIC_BASE_RPC_URL || "https://mainnet.base.org"),
 [baseSepolia.id]: http(process.env.NEXT_PUBLIC_BASE_SEPOLIA_RPC_URL || "https://sepolia.base.org"),
 },
});
```

Create app/providers.tsx to wrap your app:

```typescript
"use client";

import { ReactNode } from "react";
import { OnchainKitProvider } from "@coinbase/onchainkit";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { WagmiProvider } from "wagmi";
import { base } from "wagmi/chains";
import { config } from "@/lib/config";

const queryClient = new QueryClient();

export function Providers({ children }: { children: ReactNode }) {
 return (
 <WagmiProvider config={config}>
 <QueryClientProvider client={queryClient}>
 <OnchainKitProvider
 apiKey={process.env.NEXT_PUBLIC_ONCHAINKIT_API_KEY}
 chain={base}
 >
 {children}
 </OnchainKitProvider>
 </QueryClientProvider>
 </WagmiProvider>
 );
}
```

Update app/layout.tsx:

```typescript
import type { Metadata } from "next";
import { Inter } from "next/font/google";
import "./globals.css";
import { Providers } from "./providers";
import "@coinbase/onchainkit/styles.css";

const inter = Inter({ subsets: ["latin"] });

export const metadata: Metadata = {
 title: "My Base dApp",
 description: "Built on Base with Next.js and OnchainKit",
};

export default function RootLayout({
 children,
}: {
 children: React.ReactNode;
}) {
 return (
 <html lang="en">
 <body className={inter.className}>
 <Providers>{children}</Providers>
 </body>
 </html>
 );
}
```

## Environment Variables

Create .env.local for development:

```bash
# Base Network RPCs
NEXT_PUBLIC_BASE_RPC_URL=https://mainnet.base.org
NEXT_PUBLIC_BASE_SEPOLIA_RPC_URL=https://sepolia.base.org

# Chain ID (8453 for Base mainnet, 84532 for Sepolia)
NEXT_PUBLIC_CHAIN_ID=8453

# OnchainKit API Key (get from https://portal.cdp.coinbase.com/)
NEXT_PUBLIC_ONCHAINKIT_API_KEY=your_api_key_here

# WalletConnect Project ID (get from https://cloud.walletconnect.com/)
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id_here

# Contract Addresses
NEXT_PUBLIC_IDENTITY_REGISTRY=0x8004A169FB4a3325136EB29fA0ceB6D2e539a432
NEXT_PUBLIC_REPUTATION_REGISTRY=0x8004BAa17C55a88189AE136b182e5fdA19dE9b63

# Server-side only (no NEXT_PUBLIC prefix)
PRIVATE_KEY=0x...
BASE_RPC_URL=https://mainnet.base.org
```

Create .env.example for version control:

```bash
# Copy this file to .env.local and fill in values
NEXT_PUBLIC_BASE_RPC_URL=
NEXT_PUBLIC_BASE_SEPOLIA_RPC_URL=
NEXT_PUBLIC_CHAIN_ID=8453
NEXT_PUBLIC_ONCHAINKIT_API_KEY=
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=
NEXT_PUBLIC_IDENTITY_REGISTRY=0x8004A169FB4a3325136EB29fA0ceB6D2e539a432
NEXT_PUBLIC_REPUTATION_REGISTRY=0x8004BAa17C55a88189AE136b182e5fdA19dE9b63
PRIVATE_KEY=
BASE_RPC_URL=
```

Add to .gitignore:

```
.env*.local
.env
```

## Contract Configuration

Create lib/contracts.ts for contract addresses and ABIs:

```typescript
import { Address } from "viem";

export const CONTRACTS = {
 IDENTITY_REGISTRY: "0x8004A169FB4a3325136EB29fA0ceB6D2e539a432" as Address,
 REPUTATION_REGISTRY: "0x8004BAa17C55a88189AE136b182e5fdA19dE9b63" as Address,
 USDC: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913" as Address,
 AERODROME_ROUTER: "0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43" as Address,
} as const;

// ERC20 ABI for USDC
export const ERC20_ABI = [
 {
 inputs: [{name: "account", type: "address"}],
 name: "balanceOf",
 outputs: [{name: "", type: "uint256"}],
 stateMutability: "view",
 type: "function",
 },
 {
 inputs: [
 {name: "spender", type: "address"},
 {name: "amount", type: "uint256"},
 ],
 name: "approve",
 outputs: [{name: "", type: "bool"}],
 stateMutability: "nonpayable",
 type: "function",
 },
] as const;
```

## Next.js Configuration

Update next.config.js for Base dApp optimization:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
 reactStrictMode: true,
 webpack: (config) => {
 config.resolve.fallback = { fs: false, net: false, tls: false };
 config.externals.push("pino-pretty", "lokijs", "encoding");
 return config;
 },
};

module.exports = nextConfig;
```

## TypeScript Configuration

Verify tsconfig.json includes:

```json
{
 "compilerOptions": {
 "target": "ES2017",
 "lib": ["dom", "dom.iterable", "esnext"],
 "allowJs": true,
 "skipLibCheck": true,
 "strict": true,
 "noEmit": true,
 "esModuleInterop": true,
 "module": "esnext",
 "moduleResolution": "bundler",
 "resolveJsonModule": true,
 "isolatedModules": true,
 "jsx": "preserve",
 "incremental": true,
 "plugins": [
 {
 "name": "next"
 }
 ],
 "paths": {
 "@/*": ["./*"]
 }
 },
 "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
 "exclude": ["node_modules"]
}
```

## Running the Development Server

Start the Next.js development server:

```bash
npm run dev
```

Open http://localhost:3000 to see your app. The development server includes:
- Hot module replacement
- Fast refresh for instant updates
- Automatic compilation
- Error overlay

## Common Patterns

Create reusable hook in lib/hooks.ts:

```typescript
import { useAccount, useChainId } from "wagmi";
import { base } from "wagmi/chains";

export function useIsBaseChain() {
 const chainId = useChainId();
 return chainId === base.id;
}

export function useIsConnected() {
 const { isConnected, address } = useAccount();
 return { isConnected, address };
}
```

Create app/page.tsx with basic wallet connection:

```typescript
"use client";

import { ConnectWallet } from "@coinbase/onchainkit/wallet";
import { useAccount } from "wagmi";

export default function Home() {
 const { address, isConnected } = useAccount();

 return (
 <main className="flex min-h-screen flex-col items-center justify-center p-24">
 <h1 className="text-4xl font-bold mb-8">My Base dApp</h1>
 <ConnectWallet />
 {isConnected && (
 <div className="mt-4">
 <p>Connected: {address}</p>
 </div>
 )}
 </main>
 );
}
```

## Gotchas

**Client vs Server Components**: Use "use client" directive for components that use wagmi hooks or browser APIs. Server components cannot access window or client-side state.

**Environment Variable Prefixes**: NEXT_PUBLIC_ prefix makes variables available to the browser. Never prefix sensitive keys (private keys, server API keys) with NEXT_PUBLIC_.

**Hydration Errors**: When using wallet state, wrap conditional rendering in useEffect or use dynamic imports with { ssr: false } to prevent hydration mismatches.

**RPC Rate Limits**: Free public RPCs (https://mainnet.base.org) have rate limits. For production, use dedicated RPC providers (Alchemy, QuickNode, Infura).

**Webpack Config**: The webpack fallbacks in next.config.js are necessary to resolve Node.js modules that wagmi/viem attempt to import but don't exist in the browser.

**OnchainKit Styles**: Always import "@coinbase/onchainkit/styles.css" in your root layout for proper component styling.