# Hardhat Testing

## Overview

Hardhat provides a comprehensive testing environment using Mocha as the test runner and Chai for assertions. This lesson covers writing effective smart contract tests, using fixtures for efficient test setup, forking Base mainnet for integration testing, and generating gas reports and coverage analysis.

## Basic Test Structure with Mocha and Chai

Hardhat uses Mocha for test organization and Chai for assertions. Create `test/MyToken.test.ts`:

```typescript
import { expect } from "chai";
import { ethers } from "hardhat";
import { MyToken } from "../typechain-types";
import { SignerWithAddress } from "@nomicfoundation/hardhat-ethers/signers";

describe("MyToken", function () {
  let myToken: MyToken;
  let owner: SignerWithAddress;
  let addr1: SignerWithAddress;
  let addr2: SignerWithAddress;

  beforeEach(async function () {
    [owner, addr1, addr2] = await ethers.getSigners();
    const MyTokenFactory = await ethers.getContractFactory("MyToken");
    myToken = await MyTokenFactory.deploy();
    await myToken.waitForDeployment();
  });

  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      expect(await myToken.owner()).to.equal(owner.address);
    });

    it("Should mint initial supply to owner", async function () {
      const ownerBalance = await myToken.balanceOf(owner.address);
      const totalSupply = await myToken.totalSupply();
      expect(ownerBalance).to.equal(totalSupply);
      expect(totalSupply).to.equal(ethers.parseUnits("1000000", 18));
    });
  });

  describe("Transactions", function () {
    it("Should transfer tokens between accounts", async function () {
      const transferAmount = ethers.parseUnits("100", 18);
      await myToken.transfer(addr1.address, transferAmount);
      expect(await myToken.balanceOf(addr1.address)).to.equal(transferAmount);

      await myToken.connect(addr1).transfer(addr2.address, transferAmount);
      expect(await myToken.balanceOf(addr1.address)).to.equal(0);
      expect(await myToken.balanceOf(addr2.address)).to.equal(transferAmount);
    });

    it("Should fail if sender doesn't have enough tokens", async function () {
      const initialOwnerBalance = await myToken.balanceOf(owner.address);
      await expect(
        myToken.connect(addr1).transfer(owner.address, 1)
      ).to.be.revertedWithCustomError(myToken, "ERC20InsufficientBalance");
      expect(await myToken.balanceOf(owner.address)).to.equal(initialOwnerBalance);
    });
  });

  describe("Minting", function () {
    it("Should allow owner to mint tokens", async function () {
      const mintAmount = ethers.parseUnits("1000", 18);
      await myToken.mint(addr1.address, mintAmount);
      expect(await myToken.balanceOf(addr1.address)).to.equal(mintAmount);
    });

    it("Should fail if non-owner tries to mint", async function () {
      const mintAmount = ethers.parseUnits("1000", 18);
      await expect(
        myToken.connect(addr1).mint(addr2.address, mintAmount)
      ).to.be.revertedWithCustomError(myToken, "OwnableUnauthorizedAccount");
    });
  });
});
```

Run tests:

```bash
npx hardhat test
npx hardhat test test/MyToken.test.ts  # Run specific test file
npx hardhat test --grep "Minting"      # Run tests matching pattern
```

## Hardhat Chai Matchers

Hardhat provides custom Chai matchers for Ethereum-specific assertions:

```typescript
import { expect } from "chai";
import { ethers } from "hardhat";

describe("Advanced Matchers", function () {
  let myToken: MyToken;
  let owner: SignerWithAddress;
  let addr1: SignerWithAddress;

  beforeEach(async function () {
    [owner, addr1] = await ethers.getSigners();
    const MyTokenFactory = await ethers.getContractFactory("MyToken");
    myToken = await MyTokenFactory.deploy();
    await myToken.waitForDeployment();
  });

  it("Should emit Transfer event", async function () {
    const transferAmount = ethers.parseUnits("100", 18);
    await expect(myToken.transfer(addr1.address, transferAmount))
      .to.emit(myToken, "Transfer")
      .withArgs(owner.address, addr1.address, transferAmount);
  });

  it("Should revert with custom error", async function () {
    await expect(
      myToken.connect(addr1).mint(addr1.address, 1000)
    ).to.be.revertedWithCustomError(myToken, "OwnableUnauthorizedAccount")
      .withArgs(addr1.address);
  });

  it("Should change balance", async function () {
    const transferAmount = ethers.parseUnits("100", 18);
    await expect(
      myToken.transfer(addr1.address, transferAmount)
    ).to.changeTokenBalance(myToken, addr1, transferAmount);
  });

  it("Should change multiple balances", async function () {
    const transferAmount = ethers.parseUnits("100", 18);
    await expect(
      myToken.transfer(addr1.address, transferAmount)
    ).to.changeTokenBalances(
      myToken,
      [owner, addr1],
      [-transferAmount, transferAmount]
    );
  });

  it("Should be reverted with reason", async function () {
    await expect(
      myToken.connect(addr1).transfer(owner.address, 1)
    ).to.be.revertedWithCustomError(myToken, "ERC20InsufficientBalance");
  });
});
```

## Fixtures with loadFixture

Fixtures improve test performance by caching blockchain state. Use `loadFixture` from `@nomicfoundation/hardhat-toolbox/network-helpers`:

```typescript
import { loadFixture } from "@nomicfoundation/hardhat-toolbox/network-helpers";
import { expect } from "chai";
import { ethers } from "hardhat";

describe("MyToken with Fixtures", function () {
  async function deployTokenFixture() {
    const [owner, addr1, addr2] = await ethers.getSigners();
    const MyTokenFactory = await ethers.getContractFactory("MyToken");
    const myToken = await MyTokenFactory.deploy();
    await myToken.waitForDeployment();
    
    return { myToken, owner, addr1, addr2 };
  }

  async function deployTokenWithTransferFixture() {
    const { myToken, owner, addr1, addr2 } = await loadFixture(deployTokenFixture);
    const transferAmount = ethers.parseUnits("1000", 18);
    await myToken.transfer(addr1.address, transferAmount);
    
    return { myToken, owner, addr1, addr2, transferAmount };
  }

  describe("Deployment", function () {
    it("Should set the right owner", async function () {
      const { myToken, owner } = await loadFixture(deployTokenFixture);
      expect(await myToken.owner()).to.equal(owner.address);
    });

    it("Should mint initial supply", async function () {
      const { myToken, owner } = await loadFixture(deployTokenFixture);
      const balance = await myToken.balanceOf(owner.address);
      expect(balance).to.equal(ethers.parseUnits("1000000", 18));
    });
  });

  describe("After Transfer", function () {
    it("Should have correct balances", async function () {
      const { myToken, addr1, transferAmount } = await loadFixture(
        deployTokenWithTransferFixture
      );
      expect(await myToken.balanceOf(addr1.address)).to.equal(transferAmount);
    });

    it("Should allow further transfers", async function () {
      const { myToken, addr1, addr2, transferAmount } = await loadFixture(
        deployTokenWithTransferFixture
      );
      await myToken.connect(addr1).transfer(addr2.address, transferAmount);
      expect(await myToken.balanceOf(addr2.address)).to.equal(transferAmount);
    });
  });
});
```

## Forking Base Mainnet

Test your contracts against live Base contracts by forking mainnet:

```typescript
import { loadFixture } from "@nomicfoundation/hardhat-toolbox/network-helpers";
import { expect } from "chai";
import { ethers } from "hardhat";

describe("Integration with USDC on Base", function () {
  async function forkBaseFixture() {
    // Fork Base mainnet at specific block
    await network.provider.request({
      method: "hardhat_reset",
      params: [
        {
          forking: {
            jsonRpcUrl: "https://mainnet.base.org",
            blockNumber: 10000000,
          },
        },
      ],
    });

    const [deployer, user] = await ethers.getSigners();
    
    // Get USDC contract on Base
    const USDC_ADDRESS = "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913";
    const usdc = await ethers.getContractAt("IERC20", USDC_ADDRESS);
    
    // Impersonate whale account with USDC
    const USDC_WHALE = "0x20FE51A9229EEf2cF8Ad9E89d91CAb9312cF3b7A";
    await network.provider.request({
      method: "hardhat_impersonateAccount",
      params: [USDC_WHALE],
    });
    const whale = await ethers.getSigner(USDC_WHALE);
    
    // Fund whale with ETH for gas
    await deployer.sendTransaction({
      to: USDC_WHALE,
      value: ethers.parseEther("10"),
    });
    
    return { usdc, whale, deployer, user };
  }

  it("Should interact with USDC on Base", async function () {
    const { usdc, whale, user } = await loadFixture(forkBaseFixture);
    
    const amount = ethers.parseUnits("1000", 6); // USDC has 6 decimals
    const whaleBalance = await usdc.balanceOf(whale.address);
    expect(whaleBalance).to.be.gt(amount);
    
    // Transfer USDC from whale to user
    await usdc.connect(whale).transfer(user.address, amount);
    expect(await usdc.balanceOf(user.address)).to.equal(amount);
  });

  it("Should interact with Aerodrome Router", async function () {
    const { usdc, whale, user } = await loadFixture(forkBaseFixture);
    
    const AERODROME_ROUTER = "0xcF77a3Ba9A5CA399B7c97c74d54e5b1Beb874E43";
    const router = await ethers.getContractAt("IAerodromeRouter", AERODROME_ROUTER);
    
    // Approve router to spend USDC
    const amount = ethers.parseUnits("1000", 6);
    await usdc.connect(whale).approve(AERODROME_ROUTER, amount);
    
    // Query router for swap quote
    const WETH = "0x4200000000000000000000000000000000000006";
    const routes = [{
      from: await usdc.getAddress(),
      to: WETH,
      stable: false,
      factory: "0x420DD381b31aEf6683db6B902084cB0FFECe40Da",
    }];
    
    const amounts = await router.getAmountsOut(amount, routes);
    expect(amounts[1]).to.be.gt(0);
  });
});
```

Enable forking in `hardhat.config.ts`:

```typescript
hardhat: {
  forking: {
    url: process.env.BASE_RPC_URL || "https://mainnet.base.org",
    enabled: true,
    blockNumber: 10000000,
  },
}
```

## Gas Reporting

Enable gas reporting in `hardhat.config.ts`:

```typescript
gasReporter: {
  enabled: process.env.REPORT_GAS === "true",
  currency: "USD",
  coinmarketcap: process.env.COINMARKETCAP_API_KEY,
  token: "ETH",
  gasPrice: 21,
  outputFile: "gas-report.txt",
  noColors: true,
}
```

Run tests with gas reporting:

```bash
REPORT_GAS=true npx hardhat test
```

Output shows gas costs for each function:

```
Â·--------------------------------|---------------------------|-------------|-----------------------------Â·
|      Solc version: 0.8.24      Â·  Optimizer enabled: true  Â·  Runs: 200  Â·  Block limit: 30000000 gas  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  Methods                                                                                                â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  Contract       Â·  Method      Â·  Min        Â·  Max        Â·  Avg        Â·  # calls      Â·  usd (avg)  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  MyToken        Â·  mint        Â·      51234  Â·      68346  Â·      59790  Â·           10  Â·       1.50  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
|  MyToken        Â·  transfer    Â·      34567  Â·      51678  Â·      43122  Â·           25  Â·       1.08  â
Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·|Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·Â·
```

## Test Coverage

Generate code coverage reports:

```bash
npx hardhat coverage
```

This generates:
- `coverage/` -- HTML coverage report
- `coverage.json` -- Machine-readable coverage data

View HTML report:

```bash
open coverage/index.html
```

Coverage report shows:
- Statement coverage -- Which lines were executed
- Branch coverage -- Which conditional branches were taken
- Function coverage -- Which functions were called
- Line coverage -- Percentage of lines covered

Example output:

```
-----------------------|----------|----------|----------|----------|----------------|
File                   |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
-----------------------|----------|----------|----------|----------|----------------|
 contracts/            |      100 |      100 |      100 |      100 |                |
  MyToken.sol          |      100 |      100 |      100 |      100 |                |
-----------------------|----------|----------|----------|----------|----------------|
All files              |      100 |      100 |      100 |      100 |                |
-----------------------|----------|----------|----------|----------|----------------|
```

## Common Patterns

**Pattern 1: Time Manipulation**

```typescript
import { time } from "@nomicfoundation/hardhat-toolbox/network-helpers";

it("Should handle time-based logic", async function () {
  const { contract } = await loadFixture(deployFixture);
  
  // Increase time by 1 day
  await time.increase(86400);
  
  // Set time to specific timestamp
  await time.increaseTo(1704067200);
  
  // Get latest block timestamp
  const latestTime = await time.latest();
});
```

**Pattern 2: Snapshot and Revert**

```typescript
import { takeSnapshot } from "@nomicfoundation/hardhat-toolbox/network-helpers";

it("Should revert state changes", async function () {
  const { myToken } = await loadFixture(deployTokenFixture);
  
  const snapshot = await takeSnapshot();
  
  await myToken.transfer(addr1.address, 1000);
  expect(await myToken.balanceOf(addr1.address)).to.equal(1000);
  
  await snapshot.restore();
  expect(await myToken.balanceOf(addr1.address)).to.equal(0);
});
```

**Pattern 3: Testing Events with Multiple Arguments**

```typescript
it("Should emit complex events", async function () {
  await expect(contract.complexFunction())
    .to.emit(contract, "ComplexEvent")
    .withArgs(arg1, arg2, arg3)
    .to.emit(contract, "AnotherEvent")
    .withArgs(arg4);
});
```

## Gotchas

**Gotcha 1: Async/Await**
Always use `async`/`await` for contract interactions. Forgetting `await` causes tests to pass incorrectly. Use TypeScript strict mode to catch missing awaits.

**Gotcha 2: BigInt Comparisons**
Use Chai's equality matchers for BigInt values. Direct comparison with `===` may fail. Use `expect(value).to.equal(expectedValue)` not `expect(value === expectedValue)`.

**Gotcha 3: Forking Performance**
Forking Base mainnet makes tests slower because Hardhat fetches state from RPC. Use fixtures to cache state and run forking tests separately from unit tests.

**Gotcha 4: Impersonating Accounts**
When impersonating accounts, fund them with ETH for gas. Impersonated accounts start with zero balance and transactions will fail without gas funds.

**Gotcha 5: Coverage Accuracy**
Solidity coverage doesn't account for assembly blocks. Test assembly code separately and verify behavior manually. Some optimizer settings affect coverage reporting.