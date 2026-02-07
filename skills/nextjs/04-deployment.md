# Deploying Next.js Base dApps

## Overview

This lesson covers deploying Next.js Base dApps to production. You will learn to deploy to Vercel (recommended platform), configure environment variables for mainnet and testnet, implement network switching, optimize build performance, set up custom domains, and monitor production applications. Proper deployment ensures your dApp is fast, secure, and reliable for users.

## Vercel Deployment

Vercel (created by Next.js team) provides optimal Next.js hosting with zero configuration.

### Prerequisites

- GitHub/GitLab/Bitbucket account
- Vercel account (free tier available)
- Git repository with your Next.js project

### Connecting Repository

1. Visit https://vercel.com and sign in
2. Click "Add New" -> "Project"
3. Import your Git repository
4. Vercel auto-detects Next.js configuration
5. Configure environment variables
6. Click "Deploy"

Vercel automatically:
- Installs dependencies
- Runs build command (npm run build)
- Deploys to global CDN
- Provisions SSL certificate
- Provides preview URLs for branches

### Manual Deployment with Vercel CLI

Install Vercel CLI:

```bash
npm install -g vercel
```

Deploy from command line:

```bash
cd my-base-dapp
vercel
```

For production:

```bash
vercel --prod
```

## Environment Configuration

Managing environment variables for different environments (development, preview, production).

### Setting Environment Variables in Vercel

1. Go to project settings in Vercel dashboard
2. Navigate to "Environment Variables"
3. Add variables for each environment:

**Production Environment:**
```
NEXT_PUBLIC_BASE_RPC_URL=https://mainnet.base.org
NEXT_PUBLIC_CHAIN_ID=8453
NEXT_PUBLIC_ONCHAINKIT_API_KEY=prod_key_here
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
NEXT_PUBLIC_IDENTITY_REGISTRY=0x8004A169FB4a3325136EB29fA0ceB6D2e539a432
NEXT_PUBLIC_REPUTATION_REGISTRY=0x8004BAa17C55a88189AE136b182e5fdA19dE9b63
```

**Preview Environment (Base Sepolia):**
```
NEXT_PUBLIC_BASE_RPC_URL=https://sepolia.base.org
NEXT_PUBLIC_CHAIN_ID=84532
NEXT_PUBLIC_ONCHAINKIT_API_KEY=dev_key_here
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_project_id
```

**Development:**
Use .env.local file (not committed to git)

### Environment-Based Configuration

Create lib/config.ts with environment detection:

```typescript
import { http, createConfig } from "wagmi";
import { base, baseSepolia } from "wagmi/chains";
import { coinbaseWallet, injected } from "wagmi/connectors";

const isProduction = process.env.NODE_ENV === "production";
const chainId = parseInt(process.env.NEXT_PUBLIC_CHAIN_ID || "8453");

export const targetChain = chainId === 8453 ? base : baseSepolia;

export const config = createConfig({
 chains: [targetChain],
 connectors: [
 injected(),
 coinbaseWallet({
 appName: "My Base dApp",
 preference: "smartWalletOnly",
 }),
 ],
 transports: {
 [base.id]: http(process.env.NEXT_PUBLIC_BASE_RPC_URL),
 [baseSepolia.id]: http(process.env.NEXT_PUBLIC_BASE_SEPOLIA_RPC_URL),
 },
});

export const CONTRACTS = {
 IDENTITY_REGISTRY: process.env.NEXT_PUBLIC_IDENTITY_REGISTRY as `0x${string}`,
 REPUTATION_REGISTRY: process.env.NEXT_PUBLIC_REPUTATION_REGISTRY as `0x${string}`,
};
```

## Network Switching

Allow users to switch between Base mainnet and testnet.

### Network Switcher Component

Create app/components/network-switcher.tsx:

```typescript
"use client";

import { useSwitchChain, useChainId } from "wagmi";
import { base, baseSepolia } from "wagmi/chains";

export function NetworkSwitcher() {
 const chainId = useChainId();
 const { chains, switchChain } = useSwitchChain();

 const isMainnet = chainId === base.id;

 return (
 <div className="flex items-center space-x-2">
 <span className="text-sm font-medium">
 Network: {isMainnet ? "Base Mainnet" : "Base Sepolia"}
 </span>
 <button
 onClick={() => switchChain({ chainId: isMainnet ? baseSepolia.id : base.id })}
 className="px-3 py-1 text-sm bg-blue-600 text-white rounded hover:bg-blue-700"
 >
 Switch to {isMainnet ? "Testnet" : "Mainnet"}
 </button>
 </div>
 );
}
```

### Wrong Network Warning

```typescript
"use client";

import { useChainId, useSwitchChain } from "wagmi";
import { base } from "wagmi/chains";

export function NetworkWarning() {
 const chainId = useChainId();
 const { switchChain } = useSwitchChain();

 const expectedChainId = parseInt(process.env.NEXT_PUBLIC_CHAIN_ID || "8453");
 const isWrongNetwork = chainId !== expectedChainId;

 if (!isWrongNetwork) return null;

 return (
 <div className="bg-yellow-100 border border-yellow-400 text-yellow-800 px-4 py-3 rounded mb-4">
 <p className="font-semibold">Wrong Network</p>
 <p className="text-sm">
 Please switch to {expectedChainId === 8453 ? "Base Mainnet" : "Base Sepolia"}
 </p>
 <button
 onClick={() => switchChain({ chainId: expectedChainId })}
 className="mt-2 px-3 py-1 bg-yellow-600 text-white rounded text-sm hover:bg-yellow-700"
 >
 Switch Network
 </button>
 </div>
 );
}
```

## Build Optimization

Optimize Next.js build for production performance.

### Next.js Configuration

Update next.config.js:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
 reactStrictMode: true,
 swcMinify: true,
 compiler: {
 removeConsole: process.env.NODE_ENV === "production",
 },
 images: {
 domains: ["basescan.org", "ipfs.io"],
 formats: ["image/webp", "image/avif"],
 },
 webpack: (config, { isServer }) => {
 config.resolve.fallback = {
 ...config.resolve.fallback,
 fs: false,
 net: false,
 tls: false,
 };
 config.externals.push("pino-pretty", "lokijs", "encoding");
 return config;
 },
 experimental: {
 optimizePackageImports: ["@coinbase/onchainkit"],
 },
};

module.exports = nextConfig;
```

### Analyzing Bundle Size

Install @next/bundle-analyzer:

```bash
npm install --save-dev @next/bundle-analyzer
```

Update next.config.js:

```javascript
const withBundleAnalyzer = require("@next/bundle-analyzer")({
 enabled: process.env.ANALYZE === "true",
});

module.exports = withBundleAnalyzer(nextConfig);
```

Analyze bundle:

```bash
ANALYZE=true npm run build
```

### Code Splitting

Dynamic imports for large components:

```typescript
import dynamic from "next/dynamic";

const TokenSwap = dynamic(
 () => import("@/app/components/token-swap").then((mod) => mod.TokenSwap),
 {
 loading: () => <div>Loading swap interface...</div>,
 ssr: false,
 }
);

export default function SwapPage() {
 return (
 <main>
 <h1>Token Swap</h1>
 <TokenSwap />
 </main>
 );
}
```

## Custom Domains

Configure custom domain for your Base dApp.

### Adding Domain in Vercel

1. Go to project settings
2. Navigate to "Domains"
3. Add your domain (e.g., mybasedapp.com)
4. Configure DNS records:

**For root domain:**
```
Type: A
Name: @
Value: 76.76.21.21
```

**For www subdomain:**
```
Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

5. Wait for DNS propagation (up to 48 hours)
6. Vercel automatically provisions SSL certificate

### Base Subdomain

For Base ecosystem integration, use base subdomain:

```
Type: CNAME
Name: base
Value: cname.vercel-dns.com
```

Access at: https://base.mycompany.com

## Performance Monitoring

Monitor production dApp performance.

### Vercel Analytics

Enable in vercel.json:

```json
{
 "analytics": {
 "enable": true
 }
}
```

Install @vercel/analytics:

```bash
npm install @vercel/analytics
```

Add to app/layout.tsx:

```typescript
import { Analytics } from "@vercel/analytics/react";

export default function RootLayout({ children }: { children: React.ReactNode }) {
 return (
 <html lang="en">
 <body>
 <Providers>{children}</Providers>
 <Analytics />
 </body>
 </html>
 );
}
```

### Speed Insights

Install @vercel/speed-insights:

```bash
npm install @vercel/speed-insights
```

Add to layout:

```typescript
import { SpeedInsights } from "@vercel/speed-insights/next";

export default function RootLayout({ children }: { children: React.ReactNode }) {
 return (
 <html lang="en">
 <body>
 <Providers>{children}</Providers>
 <SpeedInsights />
 </body>
 </html>
 );
}
```

## Continuous Deployment

Automate deployments with Git integration.

### Automatic Deployments

Vercel automatically deploys:
- **Production**: Pushes to main/master branch
- **Preview**: Pull requests and other branches

Each preview gets unique URL:
```
https://my-base-dapp-git-feature-username.vercel.app
```

### Deployment Protection

Configure in Vercel project settings:
- Password protection for preview deployments
- Vercel Authentication for team access
- Branch protection rules

### CI/CD with GitHub Actions

Create .github/workflows/ci.yml:

```yaml
name: CI

on:
 push:
 branches: [main, develop]
 pull_request:
 branches: [main]

jobs:
 test:
 runs-on: ubuntu-latest
 steps:
 - uses: actions/checkout@v3
 - uses: actions/setup-node@v3
 with:
 node-version: "18"
 - run: npm ci
 - run: npm run lint
 - run: npm run type-check
 - run: npm run build

 env:
 NEXT_PUBLIC_BASE_RPC_URL: ${{ secrets.NEXT_PUBLIC_BASE_RPC_URL }}
 NEXT_PUBLIC_ONCHAINKIT_API_KEY: ${{ secrets.NEXT_PUBLIC_ONCHAINKIT_API_KEY }}
```

## Vercel Configuration File

Create vercel.json for advanced configuration:

```json
{
 "buildCommand": "npm run build",
 "devCommand": "npm run dev",
 "installCommand": "npm install",
 "framework": "nextjs",
 "regions": ["sfo1", "iad1"],
 "headers": [
 {
 "source": "/api/(.*)",
 "headers": [
 {
 "key": "Access-Control-Allow-Origin",
 "value": "*"
 },
 {
 "key": "Access-Control-Allow-Methods",
 "value": "GET, POST, PUT, DELETE, OPTIONS"
 },
 {
 "key": "Access-Control-Allow-Headers",
 "value": "Content-Type, Authorization"
 }
 ]
 }
 ],
 "redirects": [
 {
 "source": "/home",
 "destination": "/",
 "permanent": true
 }
 ],
 "crons": [
 {
 "path": "/api/cron/update-stats",
 "schedule": "0 * * * *"
 }
 ]
}
```

## Alternative Hosting Platforms

### Netlify

Deploy to Netlify:

1. Connect Git repository
2. Build command: `npm run build`
3. Publish directory: `.next`
4. Add environment variables
5. Deploy

Create netlify.toml:

```toml
[build]
 command = "npm run build"
 publish = ".next"

[[plugins]]
 package = "@netlify/plugin-nextjs"
```

### Self-Hosted

Build and run standalone:

```bash
npm run build
npm start
```

Use PM2 for process management:

```bash
npm install -g pm2
pm2 start npm --name "base-dapp" -- start
pm2 save
pm2 startup
```

Nginx reverse proxy:

```nginx
server {
 listen 80;
 server_name mybasedapp.com;

 location / {
 proxy_pass http://localhost:3000;
 proxy_http_version 1.1;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection 'upgrade';
 proxy_set_header Host $host;
 proxy_cache_bypass $http_upgrade;
 }
}
```

## Common Patterns

### Environment Detection

```typescript
export function getEnvironment() {
 if (process.env.VERCEL_ENV === "production") {
 return "production";
 }
 if (process.env.VERCEL_ENV === "preview") {
 return "preview";
 }
 return "development";
}

export function isProduction() {
 return getEnvironment() === "production";
}
```

### Feature Flags

```typescript
const FEATURES = {
 SWAP_ENABLED: process.env.NEXT_PUBLIC_ENABLE_SWAP === "true",
 MAINNET_ONLY: process.env.NEXT_PUBLIC_MAINNET_ONLY === "true",
};

export function FeatureGate({ feature, children }: { feature: keyof typeof FEATURES; children: React.ReactNode }) {
 if (!FEATURES[feature]) {
 return null;
 }
 return <>{children}</>;
}
```

## Gotchas

**Environment Variable Caching**: Vercel caches environment variables at build time. Changing them requires a new deployment.

**Serverless Function Limits**: Vercel serverless functions have size limits (50MB) and execution time limits (10s Hobby, 60s Pro).

**Edge vs Node.js Runtime**: Edge runtime is faster but has limited APIs. Use Node.js runtime for full functionality.

**Static Generation**: Pages using getStaticProps are generated at build time. Contract data may be stale -- use revalidation or client-side fetching.

**Image Optimization**: Next.js Image component requires domains to be configured in next.config.js. Add IPFS gateways and blockchain explorers.

**Cold Starts**: Serverless functions may have cold start delays. Keep critical paths client-side or use edge runtime for speed.

**Build Time**: Large dApps with many pages increase build time. Use incremental static regeneration (ISR) for frequently updated pages.