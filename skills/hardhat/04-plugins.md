# Hardhat Plugins

## Overview

Hardhat's plugin ecosystem extends its functionality with tools for verification, gas optimization, deployment management, and custom automation. This lesson covers essential plugins and how to create custom tasks for your specific workflow needs.

## hardhat-verify Plugin

The `@nomicfoundation/hardhat-verify` plugin automates contract verification on block explorers like Basescan.

### Installation and Configuration

```bash
npm install --save-dev @nomicfoundation/hardhat-verify
```

Configure in `hardhat.config.ts`:

```typescript
import "@nomicfoundation/hardhat-verify";

const config: HardhatUserConfig = {
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
};
```

### Usage

Verify a deployed contract:

```bash
npx hardhat verify --network base 0xYourContractAddress
```

Verify with constructor arguments:

```bash
npx hardhat verify --network base 0xYourContractAddress \
  "arg1" \
  "arg2" \
  123456
```

Verify using arguments file:

```typescript
// arguments.ts
module.exports = [
  "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  1000000n,
  1704067200n,
];
```

```bash
npx hardhat verify --network base \
  --constructor-args arguments.ts \
  0xYourContractAddress
```

Verify specific contract from multi-contract file:

```bash
npx hardhat verify --network base \
  --contract contracts/Token.sol:MyToken \
  0xYourContractAddress
```

## hardhat-gas-reporter Plugin

The `hardhat-gas-reporter` plugin provides detailed gas usage reports for your tests.

### Installation and Configuration

```bash
npm install --save-dev hardhat-gas-reporter
```

Configure in `hardhat.config.ts`:

```typescript
import "hardhat-gas-reporter";

const config: HardhatUserConfig = {
  gasReporter: {
    enabled: process.env.REPORT_GAS === "true",
    currency: "USD",
    coinmarketcap: process.env.COINMARKETCAP_API_KEY,
    token: "ETH",
    gasPrice: 21,
    outputFile: process.env.CI ? "gas-report.txt" : undefined,
    noColors: process.env.CI ? true : false,
    showTimeSpent: true,
    showMethodSig: true,
    excludeContracts: ["Migrations", "Mocks"],
  },
};
```

### Usage

Run tests with gas reporting:

```bash
REPORT_GAS=true npx hardhat test
```

Example output:

```
Â·-----------------------------------|---------------------------|---------------|-----------------------------Â·
|       Solc version: 0.8.24        Â·  Optimizer enabled: true  Â·  Runs: 200    Â·  Block limit: 30000000 gas  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  Methods                                                                                                      â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  Contract       Â·  Method         Â·  Min        Â·  Max        Â·  Avg          Â·  # calls      Â·  usd (avg)  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  MyToken        Â·  approve        Â·      26321  Â·      46421  Â·        36371 Â·           15  Â·       0.91  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  MyToken        Â·  mint           Â·      51234  Â·      68346  Â·        59790 Â·           10  Â·       1.50  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  MyToken        Â·  transfer       Â·      34567  Â·      51678  Â·        43122 Â·           25  Â·       1.08  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  Deployments                      Â·                           Â·  % of limit   Â·               â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  MyToken                          Â·          -  Â·          -  Â·      1234567  Â·        4.1 % Â·       30.86 â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
```

## solidity-coverage Plugin

The `solidity-coverage` plugin generates code coverage reports for Solidity contracts.

### Installation and Configuration

```bash
npm install --save-dev solidity-coverage
```

Configure in `hardhat.config.ts`:

```typescript
import "solidity-coverage";

const config: HardhatUserConfig = {
  // Coverage plugin automatically configures itself
  // Optional: customize coverage settings
  solidity: {
    version: "0.8.24",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
};
```

### Usage

Generate coverage report:

```bash
npx hardhat coverage
```

Coverage with specific network:

```bash
npx hardhat coverage --network hardhat
```

Skip specific files:

```bash
npx hardhat coverage --testfiles "test/unit/**/*.ts"
```

Configure `.solcover.js` for advanced settings:

```javascript
module.exports = {
  skipFiles: ["mocks/", "interfaces/", "test/"],
  measureStatementCoverage: true,
  measureFunctionCoverage: true,
  measureBranchCoverage: true,
  measureModifierCoverage: true,
  mocha: {
    timeout: 100000,
  },
};
```

## hardhat-deploy Plugin

The `hardhat-deploy` plugin provides advanced deployment management with tagging, dependency resolution, and multi-network support.

### Installation and Configuration

```bash
npm install --save-dev hardhat-deploy
npm install --save-dev @nomiclabs/hardhat-ethers  # Required peer dependency
```

Configure in `hardhat.config.ts`:

```typescript
import "hardhat-deploy";

const config: HardhatUserConfig = {
  namedAccounts: {
    deployer: {
      default: 0,
      base: "0xYourDeployerAddress",
    },
    admin: {
      default: 1,
      base: "0xYourAdminAddress",
    },
  },
  networks: {
    base: {
      url: "https://mainnet.base.org",
      chainId: 8453,
      accounts: process.env.PRIVATE_KEY ? [process.env.PRIVATE_KEY] : [],
    },
  },
};
```

### Usage

Create deployment script `deploy/01_deploy_token.ts`:

```typescript
import { HardhatRuntimeEnvironment } from "hardhat/types";
import { DeployFunction } from "hardhat-deploy/types";

const func: DeployFunction = async function (hre: HardhatRuntimeEnvironment) {
  const { deployments, getNamedAccounts } = hre;
  const { deploy, get } = deployments;
  const { deployer, admin } = await getNamedAccounts();

  // Deploy MyToken
  const myToken = await deploy("MyToken", {
    from: deployer,
    args: [],
    log: true,
    autoMine: true,
    waitConfirmations: 5,
  });

  console.log("MyToken deployed to:", myToken.address);

  // Verify on Basescan
  if (hre.network.name !== "hardhat" && hre.network.name !== "localhost") {
    await hre.run("verify:verify", {
      address: myToken.address,
      constructorArguments: [],
    });
  }

  // Transfer ownership
  const token = await hre.ethers.getContractAt("MyToken", myToken.address);
  const currentOwner = await token.owner();
  
  if (currentOwner === deployer && deployer !== admin) {
    console.log("Transferring ownership to admin...");
    const tx = await token.transferOwnership(admin);
    await tx.wait();
    console.log("Ownership transferred to:", admin);
  }
};

export default func;
func.tags = ["MyToken", "Token"];
func.dependencies = [];  // Add deployment dependencies here
```

Deploy contracts:

```bash
npx hardhat deploy --network base
npx hardhat deploy --network base --tags MyToken  # Deploy specific tag
npx hardhat deploy --network base --reset          # Force redeploy
```

Get deployment information:

```bash
npx hardhat export --network base --export deployments.json
```

## Custom Task Creation

Create custom Hardhat tasks for project-specific automation.

### Simple Task

Add to `hardhat.config.ts`:

```typescript
import { task } from "hardhat/config";

task("accounts", "Prints the list of accounts", async (taskArgs, hre) => {
  const accounts = await hre.ethers.getSigners();
  
  for (const account of accounts) {
    const balance = await hre.ethers.provider.getBalance(account.address);
    console.log(
      account.address,
      "-",
      hre.ethers.formatEther(balance),
      "ETH"
    );
  }
});
```

Run task:

```bash
npx hardhat accounts
npx hardhat accounts --network base
```

### Task with Parameters

```typescript
import { task } from "hardhat/config";

task("balance", "Prints an account's balance")
  .addParam("account", "The account's address")
  .setAction(async (taskArgs, hre) => {
    const balance = await hre.ethers.provider.getBalance(taskArgs.account);
    console.log(
      hre.ethers.formatEther(balance),
      "ETH"
    );
  });
```

Run with parameters:

```bash
npx hardhat balance --account 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb
```

### Advanced Task: Token Transfer

Create `tasks/transfer.ts`:

```typescript
import { task, types } from "hardhat/config";

task("transfer", "Transfer tokens to an address")
  .addParam("token", "Token contract address")
  .addParam("to", "Recipient address")
  .addParam("amount", "Amount to transfer", undefined, types.string)
  .addOptionalParam("decimals", "Token decimals", 18, types.int)
  .setAction(async (taskArgs, hre) => {
    const [signer] = await hre.ethers.getSigners();
    const token = await hre.ethers.getContractAt("IERC20", taskArgs.token);

    const amount = hre.ethers.parseUnits(taskArgs.amount, taskArgs.decimals);
    
    console.log("Transferring", taskArgs.amount, "tokens to", taskArgs.to);
    console.log("From:", signer.address);
    console.log("Token:", taskArgs.token);

    const tx = await token.transfer(taskArgs.to, amount);
    console.log("Transaction hash:", tx.hash);
    
    const receipt = await tx.wait();
    console.log("Confirmed in block:", receipt?.blockNumber);
    console.log("Gas used:", receipt?.gasUsed.toString());
  });
```

Import in `hardhat.config.ts`:

```typescript
import "./tasks/transfer";
```

Run task:

```bash
npx hardhat transfer \
  --network base \
  --token 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 \
  --to 0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb \
  --amount 100 \
  --decimals 6
```

### Task: Query Contract State

Create `tasks/query.ts`:

```typescript
import { task } from "hardhat/config";

task("query:token", "Query token information")
  .addParam("address", "Token contract address")
  .setAction(async (taskArgs, hre) => {
    const token = await hre.ethers.getContractAt("IERC20", taskArgs.address);

    try {
      const name = await token.name();
      const symbol = await token.symbol();
      const decimals = await token.decimals();
      const totalSupply = await token.totalSupply();

      console.log("Token Information:");
      console.log("Address:", taskArgs.address);
      console.log("Name:", name);
      console.log("Symbol:", symbol);
      console.log("Decimals:", decimals);
      console.log("Total Supply:", hre.ethers.formatUnits(totalSupply, decimals));

      const [signer] = await hre.ethers.getSigners();
      const balance = await token.balanceOf(signer.address);
      console.log("Your Balance:", hre.ethers.formatUnits(balance, decimals));
    } catch (error) {
      console.error("Error querying token:", error);
    }
  });

task("query:base", "Query Base-specific contracts")
  .setAction(async (taskArgs, hre) => {
    // Query USDC on Base
    const USDC = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913";
    const usdc = await hre.ethers.getContractAt("IERC20", USDC);
    
    const name = await usdc.name();
    const totalSupply = await usdc.totalSupply();
    
    console.log("USDC on Base:");
    console.log("Name:", name);
    console.log("Total Supply:", hre.ethers.formatUnits(totalSupply, 6));

    // Query Aerodrome Router
    const AERODROME_ROUTER = "0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43";
    console.log("\nAerodrome Router:", AERODROME_ROUTER);
    
    // Query EAS
    const EAS = "0x4200000000000000000000000000000000000021";
    console.log("EAS:", EAS);
  });
```

## Common Patterns

**Pattern 1: Multi-Network Task**

```typescript
task("info", "Display network information")
  .setAction(async (taskArgs, hre) => {
    console.log("Network:", hre.network.name);
    console.log("Chain ID:", (await hre.ethers.provider.getNetwork()).chainId);
    
    const [signer] = await hre.ethers.getSigners();
    console.log("Signer:", signer.address);
    
    const balance = await hre.ethers.provider.getBalance(signer.address);
    console.log("Balance:", hre.ethers.formatEther(balance), "ETH");
    
    const gasPrice = await hre.ethers.provider.getFeeData();
    console.log("Gas Price:", hre.ethers.formatUnits(gasPrice.gasPrice || 0n, "gwei"), "gwei");
  });
```

**Pattern 2: Batch Operations Task**

```typescript
task("batch:transfer", "Batch transfer tokens")
  .addParam("token", "Token address")
  .addParam("recipients", "Path to recipients JSON file")
  .setAction(async (taskArgs, hre) => {
    const recipients = JSON.parse(fs.readFileSync(taskArgs.recipients, "utf8"));
    const token = await hre.ethers.getContractAt("IERC20", taskArgs.token);
    
    for (const { address, amount } of recipients) {
      const tx = await token.transfer(address, hre.ethers.parseUnits(amount, 18));
      await tx.wait();
      console.log(`Transferred ${amount} to ${address}`);
    }
  });
```

**Pattern 3: Deployment Verification Task**

```typescript
task("verify:deployment", "Verify deployment integrity")
  .addParam("deployment", "Deployment name")
  .setAction(async (taskArgs, hre) => {
    const deployment = await hre.deployments.get(taskArgs.deployment);
    const contract = await hre.ethers.getContractAt(
      taskArgs.deployment,
      deployment.address
    );
    
    console.log("Verifying deployment:", taskArgs.deployment);
    console.log("Address:", deployment.address);
    
    // Verify contract code matches
    const onChainCode = await hre.ethers.provider.getCode(deployment.address);
    console.log("On-chain code length:", onChainCode.length);
    
    // Verify contract is functional
    try {
      await contract.name();
      console.log("Contract is functional");
    } catch (error) {
      console.error("Contract verification failed:", error);
    }
  });
```

## Gotchas

**Gotcha 1: Plugin Load Order**
Some plugins must be loaded in specific order. Always import `@nomicfoundation/hardhat-toolbox` first, then specialized plugins. Check plugin documentation for dependencies.

**Gotcha 2: Coverage Performance**
Coverage tests run slower than regular tests because they instrument code. Run coverage separately from regular test suite, especially in CI/CD pipelines.

**Gotcha 3: Gas Reporter in CI**
Gas reporter may cause CI failures if output isn't properly configured. Set `noColors: true` and `outputFile` when running in CI environments.

**Gotcha 4: Task Parameter Types**
Task parameters are strings by default. Use `types` to specify int, boolean, or other types. Forgetting to parse parameters causes runtime errors.

**Gotcha 5: hardhat-deploy vs Ignition**
Don't mix hardhat-deploy and Ignition in the same project. Choose one deployment system and stick with it. Mixing systems causes confusion about deployment state.

**Gotcha 6: Verification Rate Limits**
Basescan API has rate limits. When verifying many contracts, add delays between verification attempts or use batch verification when available. Store API keys securely in environment variables.