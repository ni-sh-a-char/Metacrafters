# Metacrafters  
**A simple token‑creating contract**

---

## Table of Contents
1. [Overview](#overview)  
2. [Installation](#installation)  
3. [Usage](#usage)  
4. [API Documentation](#api-documentation)  
5. [Examples](#examples)  
6. [Testing & Deployment](#testing--deployment)  
7. [Contributing](#contributing)  
8. [License](#license)  

---  

## Overview
`Metacrafters` is a minimal, **ERC‑20‑compatible** token contract written in Solidity. It demonstrates the essential building blocks for creating a fungible token on Ethereum (or any EVM‑compatible chain). The contract is intentionally simple so that newcomers can focus on the core concepts:

* Minting a fixed supply at deployment  
* Standard ERC‑20 functions (`balanceOf`, `transfer`, `approve`, `transferFrom`, etc.)  
* Ownership‑restricted minting (optional extension)  

The repository includes:

| Item | Description |
|------|-------------|
| `contracts/MetacraftersToken.sol` | The token contract |
| `scripts/deploy.js` | Hardhat deployment script |
| `test/MetacraftersToken.test.js` | Unit tests (Mocha/Chai) |
| `README.md` | This documentation |
| `hardhat.config.js` | Hardhat configuration (incl. Solidity ^0.8.20) |

---  

## Installation  

### Prerequisites
| Tool | Minimum version |
|------|-----------------|
| Node.js | `v18.x` (or later) |
| npm / yarn | `v8.x` (or later) |
| Hardhat | `v2.22.x` (installed via npm) |
| Solidity compiler | `0.8.20` (configured in `hardhat.config.js`) |
| Git | any recent version |

### Step‑by‑step

```bash
# 1️⃣ Clone the repository
git clone https://github.com/your‑org/Metacrafters.git
cd Metacrafters

# 2️⃣ Install dependencies
npm install          # or `yarn install`

# 3️⃣ Compile the contracts
npx hardhat compile
```

If you prefer a **Docker**‑based workflow, a ready‑made `Dockerfile` is provided:

```bash
docker build -t metacrafters .
docker run -it --rm -v $(pwd):/app metacrafters bash -c "npm install && npx hardhat compile"
```

---  

## Usage  

### 1️⃣ Deploying the contract  

The default deployment script (`scripts/deploy.js`) uses Hardhat and the `ethers` plugin.

```bash
# Deploy to a local Hardhat node
npx hardhat node               # in a separate terminal
npx hardhat run scripts/deploy.js --network localhost
```

**`scripts/deploy.js` (excerpt)**

```js
async function main() {
  const [deployer] = await ethers.getSigners();
  console.log("Deploying contract with account:", deployer.address);
  console.log("Account balance:", (await deployer.getBalance()).toString());

  const Token = await ethers.getContractFactory("MetacraftersToken");
  const token = await Token.deploy(
    "Metacrafters Token", // name
    "MCT",                // symbol
    1_000_000n * 10n ** 18n // initial supply (1M tokens, 18 decimals)
  );

  await token.waitForDeployment();
  console.log("Token deployed to:", token.target);
}
main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

> **Tip:** Change the arguments in `Token.deploy(...)` to customize name, symbol, or initial supply.

### 2️⃣ Interacting with the contract  

You can interact via:

* **Hardhat console** – quick REPL for testing
* **Ethers.js scripts** – for automation
* **Remix IDE** – for a UI‑based experience
* **Web3 front‑end** – see the *Examples* section

**Hardhat console example**

```bash
npx hardhat console --network localhost
```

```js
const token = await ethers.getContractAt("MetacraftersToken", "<deployed_address>");
await token.balanceOf(await ethers.getSigner().getAddress());
// => BigNumber { value: "1000000000000000000000000" }  // 1,000,000 tokens
await token.transfer("0xAbc...123", ethers.parseUnits("100", 18));
```

---  

## API Documentation  

Below is a concise reference for the public interface of `MetacraftersToken`. All functions follow the ERC‑20 standard unless otherwise noted.

| Function | Visibility | Parameters | Returns | Description |
|----------|------------|------------|---------|-------------|
| `constructor(string memory name_, string memory symbol_, uint256 initialSupply_)` | `public` | `name_`, `symbol_`, `initialSupply_` | — | Sets token metadata and mints `initialSupply_` to the deployer. |
| `name()` | `external view` | — | `string` | Token name. |
| `symbol()` | `external view` | — | `string` | Token symbol. |
| `decimals()` | `external view` | — | `uint8` | Fixed to `18`. |
| `totalSupply()` | `external view` | — | `uint256` | Total tokens in existence. |
| `balanceOf(address account)` | `external view` | `account` | `uint256` | Balance of `account`. |
| `transfer(address to, uint256 amount)` | `external` | `to`, `amount` | `bool` | Moves `amount` tokens from `msg.sender` to `to`. |
| `allowance(address owner, address spender)` | `external view` | `owner`, `spender` | `uint256` | Remaining tokens `spender` may spend on behalf of `owner`. |
| `approve(address spender, uint256 amount)` | `external` | `spender`, `amount` | `bool` | Sets `amount` as the allowance for `spender`. |
| `transferFrom(address from, address to, uint256 amount)` | `external` | `from`, `to`, `amount` | `bool` | Moves `amount` tokens using the allowance mechanism. |
| `increaseAllowance(address spender, uint256 addedValue)` | `external` | `spender`, `addedValue` | `bool` | Atomically increases allowance. |
| `decreaseAllowance(address spender, uint256 subtractedValue)` | `external` | `spender`, `subtractedValue` | `bool` | Atomically decreases allowance. |
| `mint(address to, uint256 amount)` *(owner‑only)* | `external onlyOwner` | `to`, `amount` | — | Creates `amount` new tokens and assigns them to `to`. |
| `burn(uint256 amount)` | `external` | `amount` | — | Destroys `amount` tokens from `msg.sender`. |
| `owner()` | `public view` | — | `address` | Returns the contract owner (deployer). |
| `transferOwnership(address newOwner)` | `public onlyOwner` | `newOwner` | — | Transfers ownership. |

### Events

| Event | Parameters | Description |
|-------|------------|-------------|
| `Transfer(address indexed from, address indexed to, uint256 value)` | `from`, `to`, `value` | Emitted on token transfer (including mint & burn). |
| `Approval(address indexed owner, address indexed spender, uint256 value)` | `owner`, `spender`, `value` | Emitted on `approve`, `increaseAllowance`, `decreaseAllowance`. |
| `OwnershipTransferred(address indexed previousOwner, address indexed newOwner)` | `previousOwner`, `newOwner` | Emitted when ownership changes. |

---  

## Examples  

### A. Simple Node.js script (Ethers.js)

```js
// scripts/transfer.js
require("dotenv").config();
const { ethers } = require("hardhat");

async function main() {
  const tokenAddress = "0xYourTokenAddress"; // replace
  const token = await ethers.getContractAt("MetacraftersToken", tokenAddress);

  const [sender, recipient] = await ethers.getSigners();

  // Show initial balances
  console.log("Sender balance:", ethers.formatUnits(await token.balanceOf(sender.address), 18));
  console.log("Recipient balance:", ethers.formatUnits(await token.balanceOf(recipient.address), 18));

  // Transfer 50 tokens
  const tx = await token.transfer(recipient.address, ethers.parseUnits("50", 18));
  await tx.wait();

  console.log("✅ Transfer complete!");
  console.log("Sender balance:", ethers.formatUnits(await token.balanceOf(sender.address), 18));
  console.log("Recipient balance:", ethers.formatUnits(await token.balanceOf(recipient.address), 18));
}

main().catch((err) => {
  console.error(err);
  process.exit(1);
});
```

Run with:

```bash
npx hardhat run scripts/transfer.js