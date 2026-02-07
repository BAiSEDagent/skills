# OnchainKit Integration in Next.js

## Overview

This lesson covers integrating Coinbase OnchainKit into your Next.js Base dApp. OnchainKit provides pre-built, production-ready React components for wallet connection, identity display, transactions, token swaps, and funding. You will learn to configure OnchainKitProvider, use core components (Identity, Wallet, Transaction, Swap, Fund), handle API key setup, and customize component behavior for your dApp needs.

## OnchainKit Provider Setup

OnchainKitProvider wraps your app and provides context to all OnchainKit components. It requires an API key from Coinbase Developer Platform.

### Getting an API Key

1. Visit https://portal.cdp.coinbase.com/
2. Sign in or create a Coinbase Developer account
3. Create a new project
4. Generate an API key for OnchainKit
5. Add to .env.local:

```bash
NEXT_PUBLIC_ONCHAINKIT_API_KEY=your_api_key_here
```

### Provider Configuration

Update app/providers.tsx with full OnchainKit configuration:

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
 config={{
 appearance: {
 mode: "auto",
 theme: "default",
 },
 }}
 >
 {children}
 </OnchainKitProvider>
 </QueryClientProvider>
 </WagmiProvider>
 );
}
```

## Wallet Components

OnchainKit provides comprehensive wallet connection and management components.

### ConnectWallet Component

Create app/components/wallet-button.tsx:

```typescript
"use client";

import {
 ConnectWallet,
 Wallet,
 WalletDropdown,
 WalletDropdownLink,
 WalletDropdownDisconnect,
} from "@coinbase/onchainkit/wallet";
import {
 Address,
 Avatar,
 Name,
 Identity,
 EthBalance,
} from "@coinbase/onchainkit/identity";

export function WalletButton() {
 return (
 <Wallet>
 <ConnectWallet>
 <Avatar className="h-6 w-6" />
 <Name />
 </ConnectWallet>
 <WalletDropdown>
 <Identity className="px-4 pt-3 pb-2" hasCopyAddressOnClick>
 <Avatar />
 <Name />
 <Address />
 <EthBalance />
 </Identity>
 <WalletDropdownLink
 icon="wallet"
 href="https://wallet.coinbase.com"
 >
 Wallet
 </WalletDropdownLink>
 <WalletDropdownDisconnect />
 </WalletDropdown>
 </Wallet>
 );
}
```

Use in app/layout.tsx:

```typescript
import { WalletButton } from "./components/wallet-button";

export default function RootLayout({
 children,
}: {
 children: React.ReactNode;
}) {
 return (
 <html lang="en">
 <body>
 <Providers>
 <header className="flex justify-between items-center p-4 border-b">
 <h1 className="text-xl font-bold">My Base dApp</h1>
 <WalletButton />
 </header>
 {children}
 </Providers>
 </body>
 </html>
 );
}
```

### Smart Wallet Support

Configure Coinbase Smart Wallet in lib/config.ts:

```typescript
import { coinbaseWallet } from "wagmi/connectors";

export const config = createConfig({
 chains: [base],
 connectors: [
 coinbaseWallet({
 appName: "My Base dApp",
 appLogoUrl: "https://example.com/logo.png",
 preference: "smartWalletOnly", // Enforce Smart Wallet
 }),
 ],
 transports: {
 [base.id]: http(),
 },
});
```

## Identity Components

OnchainKit Identity components display Basenames, avatars, and addresses.

### Identity Display

Create app/components/user-identity.tsx:

```typescript
"use client";

import {
 Identity,
 Avatar,
 Name,
 Address,
 Badge,
} from "@coinbase/onchainkit/identity";
import { base } from "viem/chains";

interface UserIdentityProps {
 address: `0x${string}`;
}

export function UserIdentity({ address }: UserIdentityProps) {
 return (
 <Identity
 address={address}
 chain={base}
 className="flex items-center space-x-3 p-4 bg-gray-100 rounded-lg"
 >
 <Avatar
 address={address}
 chain={base}
 className="h-12 w-12"
 />
 <div className="flex flex-col">
 <div className="flex items-center space-x-2">
 <Name
 address={address}
 chain={base}
 className="font-semibold text-lg"
 />
 <Badge />
 </div>
 <Address
 address={address}
 className="text-sm text-gray-600"
 />
 </div>
 </Identity>
 );
}
```

Use to display connected user:

```typescript
"use client";

import { useAccount } from "wagmi";
import { UserIdentity } from "./components/user-identity";

export default function Profile() {
 const { address } = useAccount();

 if (!address) {
 return <div>Connect wallet to view profile</div>;
 }

 return (
 <main className="container mx-auto p-8">
 <h1 className="text-3xl font-bold mb-6">Your Profile</h1>
 <UserIdentity address={address} />
 </main>
 );
}
```

### Basename Resolution

Identity components automatically resolve Basenames (e.g., "alice.base.eth"):

```typescript
import { Name } from "@coinbase/onchainkit/identity";
import { base } from "viem/chains";

export function BasenameBadge({ address }: { address: `0x${string}` }) {
 return (
 <div className="flex items-center space-x-2">
 <Name
 address={address}
 chain={base}
 className="font-medium"
 />
 <span className="text-sm text-gray-500">on Base</span>
 </div>
 );
}
```

## Transaction Components

OnchainKit Transaction components handle contract interactions with built-in UI.

### Transaction Flow

Create app/components/mint-button.tsx:

```typescript
"use client";

import {
 Transaction,
 TransactionButton,
 TransactionSponsor,
 TransactionStatus,
 TransactionStatusAction,
 TransactionStatusLabel,
} from "@coinbase/onchainkit/transaction";
import type { LifecycleStatus } from "@coinbase/onchainkit/transaction";
import { useState } from "react";
import { encodeFunctionData } from "viem";
import { base } from "viem/chains";

const NFT_CONTRACT = "0x1234567890123456789012345678901234567890";

const MINT_ABI = [
 {
 inputs: [{name: "to", type: "address"}],
 name: "mint",
 outputs: [],
 stateMutability: "payable",
 type: "function",
 },
] as const;

export function MintButton({ userAddress }: { userAddress: `0x${string}` }) {
 const [status, setStatus] = useState<LifecycleStatus>("init");

 const contracts = [
 {
 address: NFT_CONTRACT as `0x${string}`,
 abi: MINT_ABI,
 functionName: "mint",
 args: [userAddress],
 value: BigInt("10000000000000000"), // 0.01 ETH
 },
 ];

 return (
 <Transaction
 contracts={contracts}
 chainId={base.id}
 onStatus={setStatus}
 >
 <TransactionButton
 text="Mint NFT"
 className="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded"
 />
 <TransactionSponsor />
 <TransactionStatus>
 <TransactionStatusLabel />
 <TransactionStatusAction />
 </TransactionStatus>
 </Transaction>
 );
}
```

### Transaction Lifecycle

Handle transaction states:

```typescript
import type {
 LifecycleStatus,
 TransactionReceipt,
} from "@coinbase/onchainkit/transaction";

export function TransactionDemo() {
 const [txHash, setTxHash] = useState<string>("");

 const handleStatus = (status: LifecycleStatus) => {
 console.log("Transaction status:", status.statusName);

 if (status.statusName === "success" && status.statusData) {
 const receipt = status.statusData as TransactionReceipt;
 setTxHash(receipt.transactionHash);
 console.log("Transaction hash:", receipt.transactionHash);
 }

 if (status.statusName === "error") {
 console.error("Transaction error:", status.statusData);
 }
 };

 return (
 <Transaction
 contracts={[/* contract config */]}
 chainId={base.id}
 onStatus={handleStatus}
 >
 <TransactionButton text="Execute" />
 <TransactionStatus />
 {txHash && (
 <div className="mt-4">
 <a
 href={`https://basescan.org/tx/${txHash}`}
 target="_blank"
 rel="noopener noreferrer"
 className="text-blue-600 hover:underline"
 >
 View on Basescan
 </a>
 </div>
 )}
 </Transaction>
 );
}
```

## Swap Components

OnchainKit Swap provides token swapping UI with Aerodrome integration.

### SwapDefault Component

Create app/components/token-swap.tsx:

```typescript
"use client";

import { SwapDefault } from "@coinbase/onchainkit/swap";
import type { Token } from "@coinbase/onchainkit/token";
import { base } from "viem/chains";

const ETH_TOKEN: Token = {
 address: "",
 chainId: base.id,
 decimals: 18,
 name: "Ethereum",
 symbol: "ETH",
 image: "https://ethereum.org/static/eth-logo.png",
};

const USDC_TOKEN: Token = {
 address: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
 chainId: base.id,
 decimals: 6,
 name: "USD Coin",
 symbol: "USDC",
 image: "https://ethereum-optimism.github.io/data/USDC/logo.png",
};

export function TokenSwap() {
 return (
 <div className="max-w-md mx-auto p-6">
 <h2 className="text-2xl font-bold mb-4">Swap Tokens</h2>
 <SwapDefault
 from={[ETH_TOKEN]}
 to={[USDC_TOKEN]}
 />
 </div>
 );
}
```

### Custom Swap Configuration

```typescript
import {
 Swap,
 SwapAmountInput,
 SwapToggleButton,
 SwapButton,
 SwapMessage,
 SwapToast,
} from "@coinbase/onchainkit/swap";
import type { SwapError } from "@coinbase/onchainkit/swap";

export function CustomSwap() {
 const handleError = (error: SwapError) => {
 console.error("Swap error:", error.message);
 };

 return (
 <Swap
 address="0xYourAddress"
 onError={handleError}
 config={{
 maxSlippage: 3, // 3% slippage
 }}
 >
 <SwapAmountInput
 label="From"
 token={ETH_TOKEN}
 type="from"
 />
 <SwapToggleButton />
 <SwapAmountInput
 label="To"
 token={USDC_TOKEN}
 type="to"
 />
 <SwapButton />
 <SwapMessage />
 <SwapToast />
 </Swap>
 );
}
```

## Fund Components

OnchainKit Fund enables on-ramping with Coinbase Pay.

### FundButton Component

Create app/components/fund-wallet.tsx:

```typescript
"use client";

import { FundButton } from "@coinbase/onchainkit/fund";
import { useAccount } from "wagmi";

export function FundWallet() {
 const { address } = useAccount();

 if (!address) {
 return null;
 }

 return (
 <div className="p-4 bg-white rounded-lg shadow">
 <h3 className="text-lg font-semibold mb-2">Add Funds</h3>
 <p className="text-sm text-gray-600 mb-4">
 Buy crypto with Coinbase Pay
 </p>
 <FundButton
 fundingUrl={`https://pay.coinbase.com/buy/select-asset?appId=your-app-id&addresses={\"${address}\":[\"base\"]}`}
 text="Buy Crypto"
 className="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded w-full"
 />
 </div>
 );
}
```

## Checkout Components

OnchainKit Checkout for product purchases with crypto.

```typescript
import {
 Checkout,
 CheckoutButton,
 CheckoutStatus,
} from "@coinbase/onchainkit/checkout";

export function ProductCheckout() {
 const productId = "your-product-id";

 return (
 <Checkout productId={productId}>
 <CheckoutButton
 coinbaseBranded
 text="Pay with Crypto"
 />
 <CheckoutStatus />
 </Checkout>
 );
}
```

## Common Patterns

### Conditional Rendering Based on Connection

```typescript
"use client";

import { useAccount } from "wagmi";
import { ConnectWallet } from "@coinbase/onchainkit/wallet";
import { MintButton } from "./components/mint-button";

export default function MintPage() {
 const { address, isConnected } = useAccount();

 return (
 <main className="container mx-auto p-8">
 <h1 className="text-3xl font-bold mb-6">Mint NFT</h1>
 {isConnected && address ? (
 <MintButton userAddress={address} />
 ) : (
 <div className="text-center">
 <p className="mb-4">Connect your wallet to mint</p>
 <ConnectWallet />
 </div>
 )}
 </main>
 );
}
```

### Theme Customization

Customize OnchainKit theme in app/providers.tsx:

```typescript
<OnchainKitProvider
 apiKey={process.env.NEXT_PUBLIC_ONCHAINKIT_API_KEY}
 chain={base}
 config={{
 appearance: {
 mode: "light", // "light" | "dark" | "auto"
 theme: "default", // "default" | "cyberpunk" | "minimal"
 },
 }}
>
 {children}
</OnchainKitProvider>
```

## Gotchas

**API Key Required**: Most OnchainKit components require a valid API key. Without it, components will render but lack full functionality (Basename resolution, avatar fetching).

**Client Components Only**: All OnchainKit components must be used in "use client" components. They cannot be rendered server-side.

**Chain Configuration**: Ensure OnchainKitProvider chain matches your wagmi config. Mismatches cause connection issues.

**Token Lists**: For Swap components, use official token lists or manually define tokens with correct addresses for Base network.

**Smart Wallet Preference**: Setting preference: "smartWalletOnly" in coinbaseWallet connector enforces Coinbase Smart Wallet, which provides gasless transactions and better UX.

**Transaction Gas**: Even with TransactionSponsor component, not all transactions are sponsored. Ensure users have sufficient ETH for gas if sponsorship is unavailable.