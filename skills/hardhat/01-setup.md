# Hardhat Project Setup

## Overview

Hardhat is a development environment for Ethereum that provides a complete toolchain for smart contract development. This lesson covers initializing a Hardhat project, configuring it for Base (Coinbase L2), setting up TypeScript, installing essential plugins, and organizing your project structure.

## Initializing a Hardhat Project

Create a new directory and initialize Hardhat with TypeScript:

```bash
mkdir my-base-project
cd my-base-project
npm init -y
npm install --save-dev hardhat
npx hardhat init
```

When prompted, select "Create a TypeScript project" and install the sample project's dependencies. This creates:

- `hardhat.config.ts` -- Main configuration file
- `contracts/` -- Solidity source files
- `test/` -- Test files
- `ignition/modules/` -- Deployment modules
- `typechain-types/` -- Generated TypeScript types

Install the Hardhat Toolbox (includes essential plugins):

```bash
npm install --save-dev @nomicfoundation/hardhat-toolbox
```

The toolbox includes:
- `@nomicfoundation/hardhat-ethers` -- Ethers.js integration
- `@nomicfoundation/hardhat-chai-matchers` -- Chai matchers for testing
- `@nomicfoundation/hardhat-verify` -- Contract verification
- `hardhat-gas-reporter` -- Gas usage reporting
- `solidity-coverage` -- Code coverage
- `@typechain/hardhat` -- TypeScript bindings generation

## TypeScript Configuration

Hardhat creates a `tsconfig.json` file. Enhance it for strict type checking:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "outDir": "dist",
    "rootDir": "."
  },
  "include": ["./hardhat.config.ts", "./test/**/*.ts", "./ignition/**/*.ts", "./scripts/**/*.ts"],
  "files": ["./typechain-types/index.ts"]
}
```

## Configuring hardhat.config.ts for Base

Create a comprehensive configuration file for Base network:

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";
import * as dotenv from "dotenv";

dotenv.config();

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.24",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
      viaIR: true,
    },
  },
  networks: {
    base: {
      url: "https://mainnet.base.org",
      chainId: 8453,
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
      gasPrice: "auto",
    },
    baseSepolia: {
      url: "https://sepolia.base.org",
      chainId: 84532,
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
      gasPrice: "auto",
    },
    hardhat: {
      chainId: 31337,
      forking: {
        url: process.env.BASE_RPC_URL || "https://mainnet.base.org",
        enabled: process.env.FORKING === "true",
      },
    },
  },
  etherscan: {
    apiKey: {
      base: process.env.BASESCAN_API_KEY || "",
      baseSepolia: process.env.BASESCAN_API_KEY || "",
    },
    customChains: [
      {
        network: "base",
        chainId: 8453,
        urls: {
          apiURL: "https://api.basescan.org/api",
          browserURL: "https://basescan.org",
        },
      },
      {
        network: "baseSepolia",
        chainId: 84532,
        urls: {
          apiURL: "https://api-sepolia.basescan.org/api",
          browserURL: "https://sepolia.basescan.org",
        },
      },
    ],
  },
  gasReporter: {
    enabled: process.env.REPORT_GAS === "true",
    currency: "USD",
    coinmarketcap: process.env.COINMARKETCAP_API_KEY,
    token: "ETH",
  },
  typechain: {
    outDir: "typechain-types",
    target: "ethers-v6",
  },
};

export default config;
```

Create a `.env` file for sensitive configuration:

```bash
PRIVATE_KEY=0xyourprivatekeyhere
BASESCAN_API_KEY=yourapikey
BASE_RPC_URL=https://mainnet.base.org
FORKING=false
REPORT_GAS=true
COINMARKETCAP_API_KEY=yourapikey
```

Add `.env` to `.gitignore`:

```bash
node_modules
.env
cache
artifacts
artifacts-*
coverage
coverage.json
typechain-types
dist
*.log
.DS_Store
```

## Installing Essential Plugins

Install additional useful plugins:

```bash
npm install --save-dev hardhat-deploy
npm install --save-dev hardhat-contract-sizer
npm install --save-dev hardhat-abi-exporter
```

Update `hardhat.config.ts` to import plugins:

```typescript
import "hardhat-deploy";
import "hardhat-contract-sizer";
import "hardhat-abi-exporter";

// Add to config object:
{
  contractSizer: {
    alphaSort: true,
    disambiguatePaths: false,
    runOnCompile: true,
    strict: true,
  },
  abiExporter: {
    path: "./abi",
    clear: true,
    flat: true,
    spacing: 2,
  },
}
```

## Project Structure

Organize your Hardhat project with this recommended structure:

```
my-base-project/
âââ contracts/
â   âââ interfaces/
â   â   âââ IMyContract.sol
â   âââ libraries/
â   â   âââ MyLibrary.sol
â   âââ mocks/
â   â   âââ MockToken.sol
â   âââ MyContract.sol
âââ test/
â   âââ unit/
â   â   âââ MyContract.test.ts
â   âââ integration/
â       âââ MyContract.integration.test.ts
âââ ignition/
â   âââ modules/
â       âââ MyContract.ts
âââ scripts/
â   âââ deploy.ts
â   âââ interact.ts
âââ typechain-types/
âââ abi/
âââ .env
âââ .gitignore
âââ hardhat.config.ts
âââ package.json
âââ tsconfig.json
```

## Creating Your First Contract

Create `contracts/MyToken.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is ERC20, Ownable {
    constructor() ERC20("MyToken", "MTK") Ownable(msg.sender) {
        _mint(msg.sender, 1000000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }
}
```

Install OpenZeppelin contracts:

```bash
npm install @openzeppelin/contracts
```

## Compiling Contracts

Compile your contracts to generate artifacts and TypeScript types:

```bash
npx hardhat compile
```

This generates:
- `artifacts/` -- Compiled contract artifacts (ABI, bytecode)
- `cache/` -- Hardhat cache files
- `typechain-types/` -- TypeScript type definitions

Check compilation output:

```bash
npx hardhat compile --force  # Force recompilation
npx hardhat clean             # Remove artifacts and cache
```

## Common Patterns

**Pattern 1: Environment-Specific Configuration**

```typescript
const isProduction = process.env.NODE_ENV === "production";

const config: HardhatUserConfig = {
  solidity: {
    version: "0.8.24",
    settings: {
      optimizer: {
        enabled: isProduction,
        runs: isProduction ? 10000 : 200,
      },
    },
  },
};
```

**Pattern 2: Multiple Solidity Versions**

```typescript
solidity: {
  compilers: [
    {
      version: "0.8.24",
      settings: {
        optimizer: { enabled: true, runs: 200 },
      },
    },
    {
      version: "0.7.6",
      settings: {
        optimizer: { enabled: true, runs: 200 },
      },
    },
  ],
}
```

**Pattern 3: Custom RPC Endpoints**

```typescript
networks: {
  base: {
    url: process.env.ALCHEMY_BASE_URL || "https://mainnet.base.org",
    accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    timeout: 60000,
  },
}
```

## Gotchas

**Gotcha 1: Private Key Security**
Never commit private keys to version control. Always use environment variables and add `.env` to `.gitignore`. Consider using hardware wallets or key management services for production deployments.

**Gotcha 2: Network Configuration**
Base uses chain ID 8453 for mainnet and 84532 for Sepolia testnet. Incorrect chain IDs will cause deployment failures. Always verify the chain ID matches your target network.

**Gotcha 3: Gas Settings**
Base uses EIP-1559 gas pricing. Setting gasPrice: "auto" lets Hardhat calculate appropriate values. Manual gas settings may cause transactions to fail or overpay.

**Gotcha 4: TypeChain Generation**
TypeChain types are generated during compilation. If types are missing or outdated, run `npx hardhat compile` to regenerate them. Import types from `typechain-types` directory.

**Gotcha 5: Plugin Order**
Plugin import order in `hardhat.config.ts` matters. Import `@nomicfoundation/hardhat-toolbox` first, then other plugins. Some plugins modify Hardhat's behavior and must be loaded in specific order.

**Gotcha 6: Forking Configuration**
When forking Base mainnet, ensure you have a reliable RPC endpoint. Free tier RPCs may have rate limits. Use environment variable `FORKING=true` to enable forking only when needed.