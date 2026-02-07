# Next.js API Routes for Blockchain Operations

## Overview

This lesson covers building Next.js API Route Handlers for server-side blockchain operations. API routes enable secure backend logic for contract reads, signature verification, webhook processing, and data caching. You will learn to create GET/POST handlers, implement server-side contract interactions with viem, handle authentication, process webhooks, and cache blockchain data with Next.js revalidation.

## Route Handlers Basics

Next.js 14+ App Router uses Route Handlers (app/api/*/route.ts) instead of Pages Router API routes.

### Creating a Route Handler

Create app/api/hello/route.ts:

```typescript
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
 return NextResponse.json({
 message: "Hello from Base API",
 timestamp: new Date().toISOString(),
 });
}

export async function POST(request: NextRequest) {
 const body = await request.json();

 return NextResponse.json({
 received: body,
 processed: true,
 });
}
```

Route handlers support:
- GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS methods
- Request/Response objects
- Streaming responses
- Edge runtime
- Middleware integration

## Server-Side Contract Reads

Reading contract data server-side protects RPC endpoints and enables caching.

### Creating a Public Client

Create lib/viem-server.ts for server-side viem:

```typescript
import { createPublicClient, http } from "viem";
import { base } from "viem/chains";

export const publicClient = createPublicClient({
 chain: base,
 transport: http(process.env.BASE_RPC_URL || "https://mainnet.base.org"),
});
```

### Reading Contract State

Create app/api/token/balance/route.ts:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { isAddress } from "viem";
import { publicClient } from "@/lib/viem-server";

const USDC_ADDRESS = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913";

const ERC20_ABI = [
 {
 inputs: [{name: "account", type: "address"}],
 name: "balanceOf",
 outputs: [{name: "", type: "uint256"}],
 stateMutability: "view",
 type: "function",
 },
 {
 inputs: [],
 name: "decimals",
 outputs: [{name: "", type: "uint8"}],
 stateMutability: "view",
 type: "function",
 },
] as const;

export async function GET(request: NextRequest) {
 const { searchParams } = new URL(request.url);
 const address = searchParams.get("address");

 if (!address || !isAddress(address)) {
 return NextResponse.json(
 { error: "Invalid address" },
 { status: 400 }
 );
 }

 try {
 const [balance, decimals] = await Promise.all([
 publicClient.readContract({
 address: USDC_ADDRESS,
 abi: ERC20_ABI,
 functionName: "balanceOf",
 args: [address as `0x${string}`],
 }),
 publicClient.readContract({
 address: USDC_ADDRESS,
 abi: ERC20_ABI,
 functionName: "decimals",
 }),
 ]);

 const formatted = Number(balance) / Math.pow(10, decimals);

 return NextResponse.json({
 address,
 balance: balance.toString(),
 formatted,
 decimals,
 token: "USDC",
 });
 } catch (error) {
 console.error("Error reading balance:", error);
 return NextResponse.json(
 { error: "Failed to read balance" },
 { status: 500 }
 );
 }
}
```

Call from frontend:

```typescript
const response = await fetch(`/api/token/balance?address=${userAddress}`);
const data = await response.json();
console.log(`Balance: ${data.formatted} ${data.token}`);
```

### Reading Multiple Contracts

Create app/api/portfolio/route.ts:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { isAddress } from "viem";
import { publicClient } from "@/lib/viem-server";

const TOKENS = [
 {
 symbol: "USDC",
 address: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913" as `0x${string}`,
 decimals: 6,
 },
 {
 symbol: "DAI",
 address: "0x50c5725949A6F0c72E6C4a641F24049A917DB0Cb" as `0x${string}`,
 decimals: 18,
 },
];

const ERC20_ABI = [
 {
 inputs: [{name: "account", type: "address"}],
 name: "balanceOf",
 outputs: [{name: "", type: "uint256"}],
 stateMutability: "view",
 type: "function",
 },
] as const;

export async function GET(request: NextRequest) {
 const { searchParams } = new URL(request.url);
 const address = searchParams.get("address");

 if (!address || !isAddress(address)) {
 return NextResponse.json(
 { error: "Invalid address" },
 { status: 400 }
 );
 }

 try {
 const balances = await Promise.all(
 TOKENS.map(async (token) => {
 const balance = await publicClient.readContract({
 address: token.address,
 abi: ERC20_ABI,
 functionName: "balanceOf",
 args: [address as `0x${string}`],
 });

 return {
 symbol: token.symbol,
 address: token.address,
 balance: balance.toString(),
 formatted: Number(balance) / Math.pow(10, token.decimals),
 };
 })
 );

 return NextResponse.json({
 address,
 tokens: balances,
 timestamp: new Date().toISOString(),
 });
 } catch (error) {
 console.error("Error reading portfolio:", error);
 return NextResponse.json(
 { error: "Failed to read portfolio" },
 { status: 500 }
 );
 }
}
```

## Caching with Revalidation

Next.js provides built-in caching for API routes.

### Static Data with Revalidation

Create app/api/stats/route.ts:

```typescript
import { NextResponse } from "next/server";
import { publicClient } from "@/lib/viem-server";

export const revalidate = 60; // Revalidate every 60 seconds

export async function GET() {
 try {
 const blockNumber = await publicClient.getBlockNumber();
 const block = await publicClient.getBlock({ blockNumber });

 return NextResponse.json({
 blockNumber: blockNumber.toString(),
 timestamp: Number(block.timestamp),
 gasUsed: block.gasUsed.toString(),
 transactionCount: block.transactions.length,
 cached: true,
 });
 } catch (error) {
 console.error("Error fetching stats:", error);
 return NextResponse.json(
 { error: "Failed to fetch stats" },
 { status: 500 }
 );
 }
}
```

### Dynamic Revalidation

```typescript
import { NextRequest, NextResponse } from "next/server";
import { revalidateTag } from "next/cache";

export async function GET(request: NextRequest) {
 const data = await fetch("https://api.example.com/data", {
 next: { tags: ["blockchain-data"] },
 });

 return NextResponse.json(await data.json());
}

export async function POST(request: NextRequest) {
 const { tag } = await request.json();
 revalidateTag(tag);

 return NextResponse.json({ revalidated: true });
}
```

## Signature Verification

Verify SIWE (Sign-In with Ethereum) signatures server-side.

### SIWE Verification Endpoint

Install siwe:

```bash
npm install siwe
```

Create app/api/auth/verify/route.ts:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { SiweMessage } from "siwe";

export async function POST(request: NextRequest) {
 try {
 const { message, signature } = await request.json();

 const siweMessage = new SiweMessage(message);
 const fields = await siweMessage.verify({ signature });

 if (fields.data.nonce !== getExpectedNonce()) {
 return NextResponse.json(
 { error: "Invalid nonce" },
 { status: 401 }
 );
 }

 // Create session, JWT, etc.
 const sessionToken = createSessionToken(fields.data.address);

 return NextResponse.json({
 authenticated: true,
 address: fields.data.address,
 token: sessionToken,
 });
 } catch (error) {
 console.error("Verification error:", error);
 return NextResponse.json(
 { error: "Verification failed" },
 { status: 401 }
 );
 }
}

function getExpectedNonce(): string {
 // Retrieve nonce from session/database
 return "stored-nonce-value";
}

function createSessionToken(address: string): string {
 // Create JWT or session token
 return "session-token-" + address;
}
```

## Webhook Handling

Process webhooks from blockchain indexers or payment providers.

### Webhook Endpoint

Create app/api/webhooks/alchemy/route.ts:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { createHmac } from "crypto";

export async function POST(request: NextRequest) {
 try {
 const body = await request.text();
 const signature = request.headers.get("x-alchemy-signature");

 if (!verifySignature(body, signature)) {
 return NextResponse.json(
 { error: "Invalid signature" },
 { status: 401 }
 );
 }

 const payload = JSON.parse(body);

 // Process webhook
 if (payload.type === "ADDRESS_ACTIVITY") {
 await processAddressActivity(payload.event);
 }

 return NextResponse.json({ received: true });
 } catch (error) {
 console.error("Webhook error:", error);
 return NextResponse.json(
 { error: "Webhook processing failed" },
 { status: 500 }
 );
 }
}

function verifySignature(body: string, signature: string | null): boolean {
 if (!signature) return false;

 const signingKey = process.env.ALCHEMY_SIGNING_KEY;
 if (!signingKey) return false;

 const hmac = createHmac("sha256", signingKey);
 hmac.update(body);
 const digest = hmac.digest("hex");

 return digest === signature;
}

async function processAddressActivity(event: any) {
 console.log("Processing address activity:", event);
 // Update database, send notifications, etc.
}
```

## Server Actions Alternative

Next.js 14+ Server Actions provide an alternative to API routes.

### Creating Server Actions

Create app/actions/contract.ts:

```typescript
"use server";

import { publicClient } from "@/lib/viem-server";
import { isAddress } from "viem";

const IDENTITY_REGISTRY = "0x8004A169FB4a3325136EB29fA0ceB6D2e539a432";

const IDENTITY_ABI = [
 {
 inputs: [{name: "user", type: "address"}],
 name: "getIdentity",
 outputs: [
 {name: "name", type: "string"},
 {name: "verified", type: "bool"},
 ],
 stateMutability: "view",
 type: "function",
 },
] as const;

export async function getIdentity(address: string) {
 if (!isAddress(address)) {
 throw new Error("Invalid address");
 }

 const identity = await publicClient.readContract({
 address: IDENTITY_REGISTRY as `0x${string}`,
 abi: IDENTITY_ABI,
 functionName: "getIdentity",
 args: [address as `0x${string}`],
 });

 return {
 name: identity[0],
 verified: identity[1],
 address,
 };
}
```

Use in component:

```typescript
"use client";

import { useState } from "react";
import { getIdentity } from "@/app/actions/contract";

export function IdentityLookup() {
 const [identity, setIdentity] = useState<any>(null);

 const handleLookup = async (address: string) => {
 const result = await getIdentity(address);
 setIdentity(result);
 };

 return (
 <div>
 <button onClick={() => handleLookup("0x123...")}>Lookup</button>
 {identity && (
 <div>
 <p>Name: {identity.name}</p>
 <p>Verified: {identity.verified ? "Yes" : "No"}</p>
 </div>
 )}
 </div>
 );
}
```

## Authentication Middleware

Protect API routes with authentication.

### Creating Auth Middleware

Create lib/auth.ts:

```typescript
import { NextRequest, NextResponse } from "next/server";

export async function withAuth(
 request: NextRequest,
 handler: (req: NextRequest) => Promise<NextResponse>
) {
 const token = request.headers.get("authorization")?.replace("Bearer ", "");

 if (!token || !verifyToken(token)) {
 return NextResponse.json(
 { error: "Unauthorized" },
 { status: 401 }
 );
 }

 return handler(request);
}

function verifyToken(token: string): boolean {
 // Verify JWT or session token
 return token.startsWith("valid-token-");
}
```

Use in protected route:

```typescript
import { NextRequest, NextResponse } from "next/server";
import { withAuth } from "@/lib/auth";

export async function GET(request: NextRequest) {
 return withAuth(request, async (req) => {
 return NextResponse.json({ protected: "data" });
 });
}
```

## Common Patterns

### Error Handling

```typescript
import { NextRequest, NextResponse } from "next/server";

export async function GET(request: NextRequest) {
 try {
 const data = await fetchBlockchainData();
 return NextResponse.json(data);
 } catch (error) {
 console.error("API error:", error);

 return NextResponse.json(
 {
 error: "Internal server error",
 message: error instanceof Error ? error.message : "Unknown error",
 },
 { status: 500 }
 );
 }
}

async function fetchBlockchainData() {
 // Blockchain operations
 throw new Error("RPC connection failed");
}
```

### Rate Limiting

```typescript
import { NextRequest, NextResponse } from "next/server";

const rateLimits = new Map<string, number[]>();

export async function GET(request: NextRequest) {
 const ip = request.ip || "unknown";

 if (!checkRateLimit(ip, 10, 60000)) {
 return NextResponse.json(
 { error: "Rate limit exceeded" },
 { status: 429 }
 );
 }

 return NextResponse.json({ success: true });
}

function checkRateLimit(key: string, limit: number, window: number): boolean {
 const now = Date.now();
 const requests = rateLimits.get(key) || [];
 const recentRequests = requests.filter((time) => now - time < window);

 if (recentRequests.length >= limit) {
 return false;
 }

 recentRequests.push(now);
 rateLimits.set(key, recentRequests);
 return true;
}
```

## Gotchas

**Environment Variables**: Server-side API routes can access all environment variables (with or without NEXT_PUBLIC_). Never expose private keys in client-side code.

**Edge Runtime**: Route handlers default to Node.js runtime. To use Edge runtime (faster cold starts), add: export const runtime = "edge"

**Body Parsing**: Always await request.json() or request.text() before accessing body. Don't try to parse twice.

**CORS**: Add CORS headers for cross-origin requests: response.headers.set("Access-Control-Allow-Origin", "*")

**Caching**: GET handlers are cached by default. Use export const dynamic = "force-dynamic" to disable caching.

**Timeout**: Vercel serverless functions timeout after 10 seconds (Hobby) or 60 seconds (Pro). Use background jobs for long operations.