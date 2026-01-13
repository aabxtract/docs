# Writing Smart Contracts for Base

A comprehensive guide to developing, testing, and deploying smart contracts on Base, an Ethereum Layer 2 built on the OP Stack.

## Overview

Base is fully EVM-equivalent, meaning any smart contract that runs on Ethereum will run on Base without modification. This guide will help you write optimized, secure smart contracts specifically for the Base ecosystem while taking advantage of Base's low costs and fast block times.

## Table of Contents

- [Getting Started](#getting-started)
- [Development Environment Setup](#development-environment-setup)
- [Smart Contract Best Practices](#smart-contract-best-practices)
- [Base-Specific Optimizations](#base-specific-optimizations)
- [Testing Your Contracts](#testing-your-contracts)
- [Deployment Guide](#deployment-guide)
- [Verification and Monitoring](#verification-and-monitoring)
- [Security Considerations](#security-considerations)
- [Common Patterns](#common-patterns)
- [Resources](#resources)

## Getting Started

### Prerequisites

Before you begin, ensure you have:

- **Node.js v19+** installed
- **Basic Solidity knowledge** (variables, functions, events)
- **Familiarity with Ethereum concepts** (gas, transactions, addresses)
- **A wallet** with Base Sepolia ETH for testing ([get testnet ETH here](https://docs.base.org/tools/network-faucets))

### Why Build on Base?

- **Low transaction costs**: Significantly cheaper than Ethereum mainnet
- **Fast confirmations**: 2-second block times for quick user experiences
- **EVM equivalence**: Use existing Ethereum tools and contracts without changes
- **Growing ecosystem**: Access to Base-native DeFi, NFTs, and social applications
- **Ethereum security**: Inherits Ethereum's security through the OP Stack

## Development Environment Setup

### Choose Your Framework

Base supports all major Ethereum development frameworks. Choose the one that fits your workflow:

#### Foundry (Recommended)

Foundry is fast, modern, and written in Rust. Ideal for test-driven development.

```bash
# Install Foundry
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Create new project
forge init my-base-project
cd my-base-project

# Install dependencies
forge install OpenZeppelin/openzeppelin-contracts
```

#### Hardhat

Hardhat is JavaScript-based with extensive plugin ecosystem.

```bash
# Create new project
mkdir my-base-project && cd my-base-project
npm init -y
npm install --save-dev hardhat

# Initialize Hardhat
npx hardhat init
```

#### Remix

[Remix IDE](https://remix.ethereum.org/) is browser-based, perfect for quick prototyping and learning.

### Configure Base Networks

#### Foundry Configuration

Create or update `foundry.toml`:

```toml
[profile.default]
src = "src"
out = "out"
libs = ["lib"]
solc_version = "0.8.23"
optimizer = true
optimizer_runs = 200

[rpc_endpoints]
base = "https://mainnet.base.org"
base_sepolia = "https://sepolia.base.org"

[etherscan]
base = { key = "${BASESCAN_API_KEY}" }
base_sepolia = { key = "${BASESCAN_API_KEY}" }
```

#### Hardhat Configuration

Update `hardhat.config.js`:

```javascript
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: {
    version: "0.8.23",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
  networks: {
    baseSepolia: {
      url: "https://sepolia.base.org",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 84532,
    },
    base: {
      url: "https://mainnet.base.org",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 8453,
    },
  },
  etherscan: {
    apiKey: {
      base: process.env.BASESCAN_API_KEY,
      baseSepolia: process.env.BASESCAN_API_KEY,
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

### Environment Variables

Create a `.env` file in your project root:

```bash
# Network RPCs
BASE_RPC_URL=https://mainnet.base.org
BASE_SEPOLIA_RPC_URL=https://sepolia.base.org

# Deployment wallet (NEVER commit this)
PRIVATE_KEY=your_private_key_here

# Block explorer API key (for verification)
BASESCAN_API_KEY=your_basescan_api_key

# Optional: Coinbase Developer Platform API key
CDP_API_KEY=your_cdp_api_key
```

**Security Warning**: Never commit `.env` files. Add `.env` to your `.gitignore`:

```bash
echo ".env" >> .gitignore
```

## Smart Contract Best Practices

### Solidity Version

Use recent, stable Solidity versions. Avoid versions with known bugs.

```solidity
// Good: Specific stable version
pragma solidity 0.8.23;

// Acceptable: Range for library compatibility
pragma solidity ^0.8.20;

// Avoid: Outdated or too permissive
pragma solidity ^0.8.0; // Too broad
pragma solidity 0.8.4;  // Has known bugs
```

### Use OpenZeppelin Contracts

OpenZeppelin provides battle-tested, audited contract implementations.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyToken is ERC20, Ownable {
    constructor() ERC20("MyToken", "MTK") Ownable(msg.sender) {
        _mint(msg.sender, 1000000 * 10 ** decimals());
    }

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }
}
```

### Gas Optimization Patterns

While Base is cheap, gas optimization improves user experience and reduces costs further.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract GasOptimized {
    // ✅ Pack variables to save storage slots
    struct User {
        address wallet;      // 20 bytes
        uint64 balance;      // 8 bytes
        uint32 lastActive;   // 4 bytes
        // Total: 32 bytes = 1 storage slot
    }

    // ✅ Use mappings instead of arrays for lookups
    mapping(address => User) public users;

    // ✅ Cache storage variables in memory
    function processUser(address _user) external {
        User memory user = users[_user]; // Read once
        
        // Use memory variable for multiple operations
        require(user.balance > 0, "Insufficient balance");
        user.balance = user.balance / 2;
        user.lastActive = uint32(block.timestamp);
        
        users[_user] = user; // Write once
    }

    // ✅ Use custom errors instead of string reverts
    error InsufficientBalance(uint256 requested, uint256 available);

    function withdraw(uint256 amount) external {
        uint256 balance = users[msg.sender].balance;
        if (balance < amount) {
            revert InsufficientBalance(amount, balance);
        }
        users[msg.sender].balance -= amount;
    }

    // ✅ Use unchecked for safe arithmetic
    function incrementCounter(uint256 counter) public pure returns (uint256) {
        unchecked {
            return counter + 1; // Safe from overflow in this context
        }
    }
}
```

### Error Handling

Use custom errors for gas efficiency and better error messages.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract ErrorHandling {
    // ✅ Custom errors (cheaper than string reverts)
    error Unauthorized(address caller);
    error InvalidAmount(uint256 amount);
    error TransferFailed(address to, uint256 amount);

    address public owner;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert Unauthorized(msg.sender);
        }
        _;
    }

    function transfer(address to, uint256 amount) external onlyOwner {
        if (amount == 0) {
            revert InvalidAmount(amount);
        }

        (bool success, ) = to.call{value: amount}("");
        if (!success) {
            revert TransferFailed(to, amount);
        }
    }
}
```

### Events and Logging

Emit events for off-chain indexing and monitoring.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract EventLogging {
    // ✅ Index parameters for efficient filtering (max 3 indexed params)
    event TokenTransferred(
        address indexed from,
        address indexed to,
        uint256 amount,
        uint256 timestamp
    );

    event ConfigUpdated(
        string indexed parameter,
        uint256 oldValue,
        uint256 newValue
    );

    function transfer(address to, uint256 amount) external {
        // ... transfer logic ...
        
        emit TokenTransferred(msg.sender, to, amount, block.timestamp);
    }
}
```

## Base-Specific Optimizations

### Leverage Low Gas Costs

Base's low gas costs enable patterns that might be too expensive on Ethereum mainnet.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract BaseOptimized {
    // On Base, you can afford more on-chain storage
    struct DetailedTransaction {
        address from;
        address to;
        uint256 amount;
        uint256 timestamp;
        string description; // Affordable on Base, expensive on Ethereum
        bytes metadata;
    }

    DetailedTransaction[] public transactions;

    // More frequent events are affordable
    event DetailedLog(
        address indexed user,
        string action,
        bytes data,
        uint256 gasUsed
    );

    function recordTransaction(
        address to,
        uint256 amount,
        string calldata description,
        bytes calldata metadata
    ) external {
        uint256 startGas = gasleft();

        transactions.push(
            DetailedTransaction({
                from: msg.sender,
                to: to,
                amount: amount,
                timestamp: block.timestamp,
                description: description,
                metadata: metadata
            })
        );

        emit DetailedLog(
            msg.sender,
            "transaction_recorded",
            abi.encode(to, amount),
            startGas - gasleft()
        );
    }
}
```

### Account Abstraction Integration

Base has strong support for Account Abstraction (ERC-4337). Design contracts to work with smart wallets.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract AACompatible {
    // Support both EOAs and smart contract wallets
    mapping(address => uint256) public balances;

    // Use msg.sender (works for both EOAs and smart wallets)
    function deposit() external payable {
        balances[msg.sender] += msg.value;
    }

    // Verify signatures from smart wallets using ERC-1271
    function isValidSignature(
        bytes32 hash,
        bytes memory signature
    ) external view returns (bytes4) {
        // Implement ERC-1271 signature verification
        // This allows smart wallets to sign messages
        return 0x1626ba7e; // ERC-1271 magic value
    }
}
```

## Testing Your Contracts

### Unit Tests with Foundry

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

import "forge-std/Test.sol";
import "../src/MyToken.sol";

contract MyTokenTest is Test {
    MyToken token;
    address owner = address(1);
    address user = address(2);

    function setUp() public {
        vm.prank(owner);
        token = new MyToken();
    }

    function testMint() public {
        uint256 amount = 1000 * 10 ** 18;
        
        vm.prank(owner);
        token.mint(user, amount);
        
        assertEq(token.balanceOf(user), amount);
    }

    function testMintUnauthorized() public {
        vm.prank(user);
        vm.expectRevert();
        token.mint(user, 1000);
    }

    function testFuzz_Transfer(uint256 amount) public {
        vm.assume(amount > 0 && amount < token.totalSupply());
        
        vm.prank(owner);
        token.mint(user, amount);
        
        vm.prank(user);
        token.transfer(owner, amount);
        
        assertEq(token.balanceOf(owner), amount);
    }
}
```

Run tests:

```bash
# Run all tests
forge test

# Run with gas reports
forge test --gas-report

# Run specific test
forge test --match-test testMint

# Run with verbosity
forge test -vvv
```

### Fork Testing on Base

Test against live Base contracts:

```bash
# Fork Base Sepolia
forge test --fork-url https://sepolia.base.org

# Fork Base mainnet at specific block
forge test --fork-url https://mainnet.base.org --fork-block-number 10000000
```

### Integration Tests with Hardhat

```javascript
const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("MyToken", function () {
  let token;
  let owner;
  let user;

  beforeEach(async function () {
    [owner, user] = await ethers.getSigners();
    
    const MyToken = await ethers.getContractFactory("MyToken");
    token = await MyToken.deploy();
    await token.waitForDeployment();
  });

  it("Should mint tokens to user", async function () {
    const amount = ethers.parseEther("1000");
    
    await token.mint(user.address, amount);
    
    expect(await token.balanceOf(user.address)).to.equal(amount);
  });

  it("Should prevent unauthorized minting", async function () {
    await expect(
      token.connect(user).mint(user.address, 1000)
    ).to.be.reverted;
  });
});
```

## Deployment Guide

### Foundry Deployment

Create a deployment script `script/Deploy.s.sol`:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

import "forge-std/Script.sol";
import "../src/MyToken.sol";

contract DeployScript is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("PRIVATE_KEY");
        
        vm.startBroadcast(deployerPrivateKey);
        
        MyToken token = new MyToken();
        console.log("MyToken deployed to:", address(token));
        
        vm.stopBroadcast();
    }
}
```

Deploy to Base Sepolia:

```bash
# Deploy
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $BASE_SEPOLIA_RPC_URL \
  --broadcast \
  --verify

# Or use stored keystore (more secure)
forge script script/Deploy.s.sol:DeployScript \
  --rpc-url $BASE_SEPOLIA_RPC_URL \
  --account deployer \
  --broadcast \
  --verify
```

### Hardhat Deployment

Create `scripts/deploy.js`:

```javascript
const hre = require("hardhat");

async function main() {
  const MyToken = await hre.ethers.getContractFactory("MyToken");
  const token = await MyToken.deploy();
  await token.waitForDeployment();

  const address = await token.getAddress();
  console.log("MyToken deployed to:", address);

  // Wait for block confirmations before verification
  console.log("Waiting for block confirmations...");
  await token.deploymentTransaction().wait(6);

  // Verify contract
  await hre.run("verify:verify", {
    address: address,
    constructorArguments: [],
  });
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

Deploy:

```bash
npx hardhat run scripts/deploy.js --network baseSepolia
```

### Deployment Checklist

Before deploying to mainnet:

- [ ] All tests passing with >95% coverage
- [ ] Contracts audited by reputable firm (for production apps)
- [ ] Gas optimization review completed
- [ ] Deployment script tested on testnet
- [ ] Emergency pause/upgrade mechanisms tested
- [ ] Documentation complete and up-to-date
- [ ] Monitoring and alerts configured
- [ ] Deployment keys secured (hardware wallet recommended)

## Verification and Monitoring

### Verify Contracts on Basescan

#### Using Foundry

```bash
forge verify-contract \
  --chain-id 84532 \
  --compiler-version v0.8.23 \
  0x123...abc \
  src/MyToken.sol:MyToken \
  --etherscan-api-key $BASESCAN_API_KEY
```

#### Using Hardhat

```bash
npx hardhat verify --network baseSepolia 0x123...abc
```

### Monitor Contract Activity

Use these tools to monitor your deployed contracts:

- **Basescan**: View transactions, events, and state changes
- **Tenderly**: Advanced debugging and monitoring
- **The Graph**: Index and query contract events
- **OpenZeppelin Defender**: Automated monitoring and alerts

Example subgraph query:

```graphql
{
  tokenTransfers(
    first: 10
    orderBy: timestamp
    orderDirection: desc
    where: { token: "0x123...abc" }
  ) {
    from
    to
    amount
    timestamp
  }
}
```

## Security Considerations

### Common Vulnerabilities

Protect against these common issues:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract SecurityBestPractices {
    // ✅ Reentrancy protection
    uint256 private locked = 1;
    
    modifier nonReentrant() {
        require(locked == 1, "Reentrant call");
        locked = 2;
        _;
        locked = 1;
    }

    // ✅ Check-Effects-Interactions pattern
    mapping(address => uint256) public balances;

    function withdraw(uint256 amount) external nonReentrant {
        // Checks
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        // Effects
        balances[msg.sender] -= amount;
        
        // Interactions (always last)
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }

    // ✅ Input validation
    function transfer(address to, uint256 amount) external {
        require(to != address(0), "Invalid address");
        require(amount > 0, "Amount must be positive");
        require(balances[msg.sender] >= amount, "Insufficient balance");
        
        balances[msg.sender] -= amount;
        balances[to] += amount;
    }

    // ✅ Access control
    address public owner;
    
    modifier onlyOwner() {
        require(msg.sender == owner, "Not owner");
        _;
    }

    function updateConfig(uint256 newValue) external onlyOwner {
        // Only owner can call
    }
}
```

### Security Tools

Use these tools before deploying:

```bash
# Slither - Static analysis
pip install slither-analyzer
slither .

# Mythril - Symbolic execution
pip install mythril
myth analyze src/MyContract.sol

# Foundry - Invariant testing
forge test --match-test invariant
```

### Audit Preparation

Before seeking an audit:

1. **Complete documentation**: Function purposes, access controls, state changes
2. **Test coverage**: Aim for >95% line and branch coverage
3. **Known issues**: Document any known limitations or assumptions
4. **Deployment plan**: How contracts will be deployed and upgraded
5. **Threat model**: What you're protecting against

## Common Patterns

### Upgradeable Contracts

Use OpenZeppelin's upgradeable contracts pattern:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";

contract MyUpgradeableContract is 
    Initializable, 
    OwnableUpgradeable, 
    UUPSUpgradeable 
{
    uint256 public value;

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize() public initializer {
        __Ownable_init(msg.sender);
        __UUPSUpgradeable_init();
        value = 0;
    }

    function setValue(uint256 newValue) external onlyOwner {
        value = newValue;
    }

    function _authorizeUpgrade(address newImplementation) 
        internal 
        override 
        onlyOwner 
    {}
}
```

### Factory Pattern

Deploy multiple contract instances efficiently:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract ChildContract {
    address public owner;
    string public name;

    constructor(address _owner, string memory _name) {
        owner = _owner;
        name = _name;
    }
}

contract Factory {
    event ContractCreated(address indexed creator, address contractAddress);
    
    address[] public deployedContracts;
    mapping(address => address[]) public userContracts;

    function createContract(string calldata name) external returns (address) {
        ChildContract newContract = new ChildContract(msg.sender, name);
        address contractAddress = address(newContract);
        
        deployedContracts.push(contractAddress);
        userContracts[msg.sender].push(contractAddress);
        
        emit ContractCreated(msg.sender, contractAddress);
        return contractAddress;
    }

    function getUserContracts(address user) 
        external 
        view 
        returns (address[] memory) 
    {
        return userContracts[user];
    }
}
```

### Multi-Signature Wallet

Require multiple approvals for critical operations:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.23;

contract MultiSig {
    event Submission(uint256 indexed txId);
    event Confirmation(address indexed owner, uint256 indexed txId);
    event Execution(uint256 indexed txId);

    address[] public owners;
    mapping(address => bool) public isOwner;
    uint256 public required;

    struct Transaction {
        address to;
        uint256 value;
        bytes data;
        bool executed;
        uint256 confirmations;
    }

    mapping(uint256 => Transaction) public transactions;
    mapping(uint256 => mapping(address => bool)) public confirmations;
    uint256 public transactionCount;

    constructor(address[] memory _owners, uint256 _required) {
        require(_owners.length > 0, "Owners required");
        require(_required > 0 && _required <= _owners.length, "Invalid requirement");

        for (uint256 i = 0; i < _owners.length; i++) {
            address owner = _owners[i];
            require(owner != address(0), "Invalid owner");
            require(!isOwner[owner], "Duplicate owner");

            isOwner[owner] = true;
            owners.push(owner);
        }

        required = _required;
    }

    function submitTransaction(
        address to,
        uint256 value,
        bytes calldata data
    ) external returns (uint256) {
        require(isOwner[msg.sender], "Not owner");

        uint256 txId = transactionCount++;
        transactions[txId] = Transaction({
            to: to,
            value: value,
            data: data,
            executed: false,
            confirmations: 0
        });

        emit Submission(txId);
        return txId;
    }

    function confirmTransaction(uint256 txId) external {
        require(isOwner[msg.sender], "Not owner");
        require(!transactions[txId].executed, "Already executed");
        require(!confirmations[txId][msg.sender], "Already confirmed");

        confirmations[txId][msg.sender] = true;
        transactions[txId].confirmations++;

        emit Confirmation(msg.sender, txId);

        if (transactions[txId].confirmations >= required) {
            executeTransaction(txId);
        }
    }

    function executeTransaction(uint256 txId) internal {
        Transaction storage txn = transactions[txId];
        require(!txn.executed, "Already executed");

        txn.executed = true;
        (bool success, ) = txn.to.call{value: txn.value}(txn.data);
        require(success, "Execution failed");

        emit Execution(txId);
    }
}
```

## Resources

### Official Documentation

- [Base Developer Docs](https://docs.base.org)
- [OP Stack Specifications](https://github.com/ethereum-optimism/specs)
- [Base GitHub](https://github.com/base-org)

### Development Tools

- [Foundry Book](https://book.getfoundry.sh/)
- [Hardhat Documentation](https://hardhat.org/docs)
- [OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts)
- [Remix IDE](https://remix.ethereum.org/)

### Security Resources

- [Smart Contract Security Best Practices](https://consensys.github.io/smart-contract-best-practices/)
- [OpenZeppelin Security Audits](https://blog.openzeppelin.com/security-audits)
- [Ethernaut - Security Challenges](https://ethernaut.openzeppelin.com/)
- [Slither Documentation](https://github.com/crytic/slither)

### Community

- [Base Discord](https://discord.gg/buildonbase)
- [Base Twitter](https://twitter.com/base)
- [Ethereum Stack Exchange](https://ethereum.stackexchange.com/)

### Example Projects

- [Base Examples Repository](https://github.com/base-org/base-examples)
- [OnchainKit Templates](https://github.com/coinbase/onchainkit)
- [Base Starter Kits](https://docs.base.org/tools/build-on-base)

## Contributing

Found an error or want to add more examples? Contributions are welcome!

1. Fork this repository
2. Create a feature branch
3. Submit a pull request with your improvements

## License

This documentation is provided under the MIT License. Code examples are provided as-is for educational purposes.

---

**Need help?** Join the [Base Discord](https://discord.gg/buildonbase) or open an issue in this repository.
