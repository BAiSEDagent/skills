# Hardhat Deployment

## Overview

Hardhat provides multiple deployment methods, with Hardhat Ignition being the recommended approach for modern projects. This lesson covers deploying contracts to Base, verifying on Basescan, managing deployment artifacts, and implementing proxy patterns.

## Hardhat Ignition Deployment Modules

Hardhat Ignition provides declarative, reproducible deployments. Create `ignition/modules/MyToken.ts`:

```typescript
import { buildModule } from "@nomicfoundation/hardhat-ignition/modules";

const MyTokenModule = buildModule("MyTokenModule", (m) => {
  // Deploy MyToken contract
  const myToken = m.contract("MyToken", []);

  return { myToken };
});

export default MyTokenModule;
```

Deploy to Base:

```bash
npx hardhat ignition deploy ignition/modules/MyToken.ts --network base
```

Ignition creates deployment artifacts in `ignition/deployments/`:

```
ignition/deployments/chain-8453/
âââ deployed_addresses.json
âââ journal.jsonl
âââ artifacts/
    âââ MyTokenModule#MyToken.json
```

## Complex Deployment with Parameters

Create parameterized deployment for contracts with constructor arguments:

```typescript
import { buildModule } from "@nomicfoundation/hardhat-ignition/modules";

const TokenSaleModule = buildModule("TokenSaleModule", (m) => {
  // Parameters with defaults
  const tokenPrice = m.getParameter("tokenPrice", 1000000n); // 1 USDC
  const saleStart = m.getParameter("saleStart", 1704067200n);
  const saleDuration = m.getParameter("saleDuration", 604800n); // 1 week
  const usdcAddress = m.getParameter(
    "usdcAddress",
    "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"
  );

  // Deploy token first
  const myToken = m.contract("MyToken", []);

  // Deploy sale contract with dependencies
  const tokenSale = m.contract("TokenSale", [
    myToken,
    usdcAddress,
    tokenPrice,
    saleStart,
    saleDuration,
  ]);

  // Transfer tokens to sale contract
  const transferAmount = 1000000n * 10n ** 18n;
  m.call(myToken, "transfer", [tokenSale, transferAmount]);

  // Transfer ownership to sale contract
  m.call(myToken, "transferOwnership", [tokenSale]);

  return { myToken, tokenSale };
});

export default TokenSaleModule;
```

Create parameter file `ignition/parameters/base-mainnet.json`:

```json
{
  "TokenSaleModule": {
    "tokenPrice": "1000000",
    "saleStart": "1704067200",
    "saleDuration": "604800",
    "usdcAddress": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"
  }
}
```

Deploy with parameters:

```bash
npx hardhat ignition deploy ignition/modules/TokenSale.ts \
  --network base \
  --parameters ignition/parameters/base-mainnet.json
```

## Verification on Basescan

Configure verification in `hardhat.config.ts`:

```typescript
import { HardhatUserConfig } from "hardhat/config";
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

export default config;
```

Ignition automatically verifies contracts during deployment:

```bash
npx hardhat ignition deploy ignition/modules/MyToken.ts \
  --network base \
  --verify
```

Manually verify a deployed contract:

```bash
npx hardhat verify --network base 0xYourContractAddress
```

Verify with constructor arguments:

```bash
npx hardhat verify --network base 0xYourContractAddress \
  "Constructor Arg 1" \
  1000000 \
  0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913
```

For complex constructor arguments, create `arguments.js`:

```javascript
module.exports = [
  "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  1000000n,
  1704067200n,
  604800n,
];
```

Verify with arguments file:

```bash
npx hardhat verify --network base \
  --constructor-args arguments.js \
  0xYourContractAddress
```

## Traditional Deployment Scripts

For scenarios where Ignition isn't suitable, use deployment scripts. Create `scripts/deploy.ts`:

```typescript
import { ethers } from "hardhat";
import * as fs from "fs";
import * as path from "path";

async function main() {
  const [deployer] = await ethers.getSigners();
  
  console.log("Deploying contracts with account:", deployer.address);
  console.log("Account balance:", (await ethers.provider.getBalance(deployer.address)).toString());

  // Deploy MyToken
  console.log("\nDeploying MyToken...");
  const MyTokenFactory = await ethers.getContractFactory("MyToken");
  const myToken = await MyTokenFactory.deploy();
  await myToken.waitForDeployment();
  const myTokenAddress = await myToken.getAddress();
  console.log("MyToken deployed to:", myTokenAddress);

  // Deploy TokenSale
  console.log("\nDeploying TokenSale...");
  const USDC_ADDRESS = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913";
  const tokenPrice = 1000000n; // 1 USDC
  const saleStart = BigInt(Math.floor(Date.now() / 1000) + 3600); // 1 hour from now
  const saleDuration = 604800n; // 1 week

  const TokenSaleFactory = await ethers.getContractFactory("TokenSale");
  const tokenSale = await TokenSaleFactory.deploy(
    myTokenAddress,
    USDC_ADDRESS,
    tokenPrice,
    saleStart,
    saleDuration
  );
  await tokenSale.waitForDeployment();
  const tokenSaleAddress = await tokenSale.getAddress();
  console.log("TokenSale deployed to:", tokenSaleAddress);

  // Transfer tokens to sale contract
  console.log("\nTransferring tokens to TokenSale...");
  const transferAmount = ethers.parseUnits("1000000", 18);
  const transferTx = await myToken.transfer(tokenSaleAddress, transferAmount);
  await transferTx.wait();
  console.log("Transferred", ethers.formatUnits(transferAmount, 18), "tokens");

  // Transfer ownership
  console.log("\nTransferring ownership...");
  const transferOwnershipTx = await myToken.transferOwnership(tokenSaleAddress);
  await transferOwnershipTx.wait();
  console.log("Ownership transferred to TokenSale");

  // Save deployment addresses
  const deploymentInfo = {
    network: "base",
    chainId: 8453,
    deployer: deployer.address,
    timestamp: new Date().toISOString(),
    contracts: {
      MyToken: myTokenAddress,
      TokenSale: tokenSaleAddress,
    },
  };

  const deploymentsDir = path.join(__dirname, "..", "deployments");
  if (!fs.existsSync(deploymentsDir)) {
    fs.mkdirSync(deploymentsDir);
  }

  const deploymentPath = path.join(deploymentsDir, "base-mainnet.json");
  fs.writeFileSync(deploymentPath, JSON.stringify(deploymentInfo, null, 2));
  console.log("\nDeployment info saved to:", deploymentPath);

  // Verify contracts
  console.log("\nVerifying contracts on Basescan...");
  console.log("Run these commands manually:");
  console.log(`npx hardhat verify --network base ${myTokenAddress}`);
  console.log(`npx hardhat verify --network base ${tokenSaleAddress} ${myTokenAddress} ${USDC_ADDRESS} ${tokenPrice} ${saleStart} ${saleDuration}`);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Run deployment script:

```bash
npx hardhat run scripts/deploy.ts --network base
```

## Proxy Deployments

Deploy upgradeable contracts using UUPS proxy pattern with Ignition:

```typescript
import { buildModule } from "@nomicfoundation/hardhat-ignition/modules";

const MyTokenProxyModule = buildModule("MyTokenProxyModule", (m) => {
  // Deploy implementation
  const implementation = m.contract("MyTokenUpgradeable", []);

  // Encode initialization data
  const initializeData = m.encodeFunctionCall(implementation, "initialize", []);

  // Deploy proxy
  const proxy = m.contract("ERC1967Proxy", [implementation, initializeData]);

  // Return proxy as MyTokenUpgradeable type
  const myToken = m.contractAt("MyTokenUpgradeable", proxy);

  return { myToken, implementation, proxy };
});

export default MyTokenProxyModule;
```

Create upgradeable contract `contracts/MyTokenUpgradeable.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts-upgradeable/token/ERC20/ERC20Upgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

contract MyTokenUpgradeable is ERC20Upgradeable, OwnableUpgradeable, UUPSUpgradeable {
    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize() public initializer {
        __ERC20_init("MyToken", "MTK");
        __Ownable_init(msg.sender);
        __UUPSUpgradeable_init();
        
        _mint(msg.sender, 1000000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }

    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
}
```

Deploy proxy:

```bash
npx hardhat ignition deploy ignition/modules/MyTokenProxy.ts --network base --verify
```

## Managing Deployment Artifacts

Create utility script `scripts/get-deployment.ts`:

```typescript
import * as fs from "fs";
import * as path from "path";

function getDeployment(network: string) {
  const deploymentPath = path.join(__dirname, "..", "deployments", `${network}.json`);
  
  if (!fs.existsSync(deploymentPath)) {
    throw new Error(`Deployment file not found for network: ${network}`);
  }
  
  const deploymentData = fs.readFileSync(deploymentPath, "utf8");
  return JSON.parse(deploymentData);
}

function getIgnitionDeployment(network: string, moduleName: string) {
  const chainId = network === "base" ? "chain-8453" : "chain-84532";
  const deploymentPath = path.join(
    __dirname,
    "..",
    "ignition",
    "deployments",
    chainId,
    "deployed_addresses.json"
  );
  
  if (!fs.existsSync(deploymentPath)) {
    throw new Error(`Ignition deployment not found for ${network}`);
  }
  
  const deploymentData = fs.readFileSync(deploymentPath, "utf8");
  const addresses = JSON.parse(deploymentData);
  
  return addresses;
}

// Usage
const network = process.argv[2] || "base";
const deployment = getIgnitionDeployment(network, "MyTokenModule");
console.log("Deployed contracts:", JSON.stringify(deployment, null, 2));
```

## Interaction Scripts

Create script to interact with deployed contracts:

```typescript
import { ethers } from "hardhat";
import * as fs from "fs";
import * as path from "path";

async function main() {
  const network = "base";
  
  // Load deployment addresses
  const deploymentPath = path.join(__dirname, "..", "ignition", "deployments", "chain-8453", "deployed_addresses.json");
  const addresses = JSON.parse(fs.readFileSync(deploymentPath, "utf8"));
  const myTokenAddress = addresses["MyTokenModule#MyToken"];

  console.log("Interacting with MyToken at:", myTokenAddress);

  // Get contract instance
  const myToken = await ethers.getContractAt("MyToken", myTokenAddress);

  // Query contract
  const name = await myToken.name();
  const symbol = await myToken.symbol();
  const totalSupply = await myToken.totalSupply();
  
  console.log("Name:", name);
  console.log("Symbol:", symbol);
  console.log("Total Supply:", ethers.formatUnits(totalSupply, 18));

  // Get signer
  const [signer] = await ethers.getSigners();
  const balance = await myToken.balanceOf(signer.address);
  console.log("Your balance:", ethers.formatUnits(balance, 18));

  // Execute transaction
  const recipient = "0x0000000000000000000000000000000000000001";
  const amount = ethers.parseUnits("100", 18);
  
  console.log(`\nTransferring ${ethers.formatUnits(amount, 18)} tokens to ${recipient}...`);
  const tx = await myToken.transfer(recipient, amount);
  console.log("Transaction hash:", tx.hash);
  
  const receipt = await tx.wait();
  console.log("Transaction confirmed in block:", receipt?.blockNumber);
  console.log("Gas used:", receipt?.gasUsed.toString());
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

## Common Patterns

**Pattern 1: Multi-Step Deployment**

```typescript
const ComplexModule = buildModule("ComplexModule", (m) => {
  // Deploy contracts in order
  const token = m.contract("Token", []);
  const vault = m.contract("Vault", [token]);
  const strategy = m.contract("Strategy", [vault]);
  
  // Configure contracts
  m.call(vault, "setStrategy", [strategy]);
  m.call(token, "approve", [vault, ethers.MaxUint256]);
  
  return { token, vault, strategy };
});
```

**Pattern 2: Conditional Deployment**

```typescript
const ConditionalModule = buildModule("ConditionalModule", (m) => {
  const useExisting = m.getParameter("useExistingToken", false);
  const existingTokenAddress = m.getParameter("existingTokenAddress", "");
  
  const token = useExisting
    ? m.contractAt("Token", existingTokenAddress)
    : m.contract("Token", []);
  
  return { token };
});
```

**Pattern 3: Deployment with Verification Retry**

```typescript
async function deployAndVerify() {
  const contract = await deploy();
  
  let verified = false;
  let attempts = 0;
  
  while (!verified && attempts < 3) {
    try {
      await hre.run("verify:verify", {
        address: await contract.getAddress(),
        constructorArguments: [],
      });
      verified = true;
    } catch (error) {
      attempts++;
      console.log(`Verification attempt ${attempts} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, 10000));
    }
  }
}
```

## Gotchas

**Gotcha 1: Constructor Arguments Encoding**
When verifying contracts with complex constructor arguments (structs, arrays), encode them correctly. Use `arguments.js` file with proper JavaScript types, not Solidity syntax.

**Gotcha 2: Proxy Verification**
When deploying proxies, verify both the implementation and proxy contracts. Basescan may not automatically detect the implementation behind a proxy.

**Gotcha 3: Gas Estimation**
Base gas prices are typically low but can spike. Always check current gas prices before deployment and set appropriate limits in transactions. Use `gasPrice: "auto"` in network config.

**Gotcha 4: Nonce Management**
When deploying multiple contracts rapidly, nonce conflicts can occur. Ignition handles this automatically, but manual scripts should wait for transaction confirmation before proceeding.

**Gotcha 5: Deployment Artifacts**
Ignition stores deployment artifacts in network-specific directories. Don't manually edit these files -- use Ignition commands to manage deployments. Commit deployment artifacts to version control for reproducibility.