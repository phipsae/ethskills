---
name: tools
description: Current Ethereum development tools, frameworks, libraries, RPCs, and block explorers. What actually works today for building on Ethereum. Includes tool discovery for AI agents — MCPs, abi.ninja, Foundry, Scaffold-ETH 2, Hardhat, and more. Use when setting up a dev environment, choosing tools, or when an agent needs to discover what's available.
---

# Ethereum Development Tools

## Frameworks

### Scaffold-ETH 2
- **What:** Full-stack Ethereum development toolkit. Solidity + Next.js + Foundry.
- **Best for:** Rapid prototyping, building full dApps with UI, deploying to IPFS
- **Setup:** `npx create-eth@latest`
- **Docs:** https://docs.scaffoldeth.io/
- **UI Components:** https://ui.scaffoldeth.io/
- **Key feature:** Auto-generates TypeScript types from your contracts. Scaffold hooks make contract interaction trivial.
- **Deploy:** `yarn ipfs` deploys to BuidlGuidl IPFS
- **Why it's great:** Complete starter that shows you how the pieces fit together — from Solidity to React UI. Not just a template, but an educational framework that teaches modern web3 patterns.

### Foundry (Recommended for 2026)
- **What:** Blazing fast Solidity toolkit. Compile, test, deploy, interact — all from CLI.
- **Tools:** 
  - `forge` — build/test/deploy
  - `cast` — interact with contracts and chains
  - `anvil` — local Ethereum node
  - `chisel` — Solidity REPL (interactive testing)
- **Best for:** Contract development, testing, scripting, CI/CD
- **Install:** `curl -L https://foundry.paradigm.xyz | bash && foundryup`
- **Docs:** https://book.getfoundry.sh/

**Why Foundry is winning:**
- **10-100x faster tests** than Hardhat (written in Rust)
- **Write tests in Solidity** (not JavaScript) — same language as contracts
- **Built-in fuzzing** — automatically tests thousands of input combinations
- **Mainnet forking** — test against real deployed contracts locally
- **Gas profiling** — see exactly where gas is spent
- **Cheatcodes** — powerful testing utilities (`vm.prank`, `vm.warp`, `vm.deal`)

**Common Foundry Commands:**
```bash
# Create new project
forge init my-project

# Install dependencies
forge install OpenZeppelin/openzeppelin-contracts

# Run tests (with gas report and verbosity)
forge test --gas-report -vvv

# Deploy contract
forge create src/MyContract.sol:MyContract --rpc-url $RPC_URL --private-key $PRIVATE_KEY

# Verify on Etherscan
forge verify-contract 0xYourAddress src/MyContract.sol:MyContract --etherscan-api-key $KEY

# Run script
forge script script/Deploy.s.sol --rpc-url $RPC_URL --broadcast
```

### Hardhat
- **What:** Established JavaScript/TypeScript Ethereum development environment
- **Best for:** Teams coming from JavaScript, extensive plugin ecosystem
- **Setup:** `npx hardhat init`
- **Docs:** https://hardhat.org/docs
- **Note:** Foundry is generally preferred for new projects due to speed and Solidity-native testing, but Hardhat has a larger plugin ecosystem and is more familiar to JavaScript developers.

**Key Hardhat Plugins:**
- `hardhat-deploy` — advanced deployment management
- `hardhat-gas-reporter` — gas usage statistics
- `hardhat-contract-sizer` — check contract sizes
- `hardhat-etherscan` — automated verification
- `@typechain/hardhat` — TypeScript bindings for contracts

**When to choose Hardhat over Foundry:**
- Team is JavaScript-heavy and uncomfortable with Solidity testing
- Need specific plugins that don't exist in Foundry ecosystem
- Existing codebase is already Hardhat

## Libraries

### ethers.js (v6)
- **What:** The most widely used Ethereum JavaScript library
- **Install:** `npm install ethers`
- **Docs:** https://docs.ethers.org/v6/
- **Use for:** Wallet creation, contract interaction, transaction building, ENS resolution

**Common patterns:**
```javascript
import { ethers } from "ethers";

// Connect to provider
const provider = new ethers.JsonRpcProvider("https://eth.llamarpc.com");

// Get balance
const balance = await provider.getBalance("vitalik.eth");

// Connect to contract
const contract = new ethers.Contract(address, abi, provider);
const result = await contract.someFunction();

// Send transaction
const signer = new ethers.Wallet(privateKey, provider);
const tx = await contract.connect(signer).someFunction();
await tx.wait();
```

### viem (Modern Alternative)
- **What:** Modern, lightweight alternative to ethers.js. TypeScript-first.
- **Install:** `npm install viem`
- **Docs:** https://viem.sh
- **Use for:** Same as ethers.js but with better TypeScript support and smaller bundle size

**Why viem is gaining traction:**
- **Smaller bundle size** — 40-80% smaller than ethers.js
- **Better TypeScript** — type safety throughout
- **Modular** — only import what you need
- **Performance** — faster than ethers.js
- **Better DX** — cleaner API design

**Common patterns:**
```typescript
import { createPublicClient, createWalletClient, http } from "viem";
import { mainnet } from "viem/chains";

// Read-only client
const publicClient = createPublicClient({ 
  chain: mainnet, 
  transport: http() 
});

const balance = await publicClient.getBalance({ 
  address: "0x..." 
});

// Write client
const walletClient = createWalletClient({
  chain: mainnet,
  transport: http()
});

const hash = await walletClient.writeContract({
  address: contractAddress,
  abi: contractAbi,
  functionName: 'transfer',
  args: [to, amount]
});
```

### wagmi
- **What:** React hooks for Ethereum. Built on viem.
- **Best for:** React frontends that need wallet connection and contract interaction
- **Docs:** https://wagmi.sh
- **Note:** Scaffold-ETH 2 wraps wagmi with its own hooks — use those instead of raw wagmi when using SE2

**Key hooks:**
```typescript
import { useAccount, useBalance, useContractRead, useContractWrite } from 'wagmi';

// Get connected wallet
const { address, isConnected } = useAccount();

// Read contract
const { data } = useContractRead({
  address: contractAddress,
  abi: contractAbi,
  functionName: 'balanceOf',
  args: [address]
});

// Write to contract
const { write } = useContractWrite({
  address: contractAddress,
  abi: contractAbi,
  functionName: 'transfer'
});
```

### OpenZeppelin Contracts
- **What:** Battle-tested smart contract library. ERC-20, ERC-721, access control, upgrades, etc.
- **Install:** `forge install OpenZeppelin/openzeppelin-contracts`
- **Docs:** https://docs.openzeppelin.com/contracts
- **Use for:** Don't write your own token contracts from scratch. Use OpenZeppelin.

**Key components:**
- **Token standards:** ERC20, ERC721, ERC1155
- **Access control:** Ownable, AccessControl, AccessControlEnumerable
- **Security:** ReentrancyGuard, Pausable, PullPayment
- **Upgrades:** UUPS, Transparent, Beacon proxies
- **Utilities:** Address, Arrays, Strings, Math, SafeCast

**Always use OpenZeppelin for:**
- Token contracts (don't roll your own!)
- Access control patterns
- Security guards (reentrancy protection)
- Standard implementations

## Interaction Tools

### abi.ninja (Essential for Agents)
- **URL:** https://abi.ninja
- **What:** Interact with any verified smart contract directly in the browser. No code needed.
- **How:** Paste a contract address → it fetches the ABI from Etherscan → gives you a UI to call any function
- **Why it's great:** 
  - Zero setup — just paste an address
  - Auto-fetches ABIs from Etherscan/Blockscout
  - Support for multiple networks
  - Can connect wallet for write operations
  - Perfect for contract exploration and debugging
- **Agent use case:** An agent can use this via browser automation to interact with contracts without writing scripts
- **Current state (Feb 2026):** Actively maintained, supports Ethereum mainnet and all major L2s (Arbitrum, Optimism, Base, Polygon, zkSync)

**When to use abi.ninja:**
- Quick contract testing
- Exploring unfamiliar contracts
- Debugging contract interactions
- Demonstrating contract functionality to non-developers
- Agent-driven contract calls via browser automation

### Foundry cast (CLI Power Tool)
- **What:** Command-line tool for interacting with Ethereum. Part of Foundry suite.
- **Why agents should use it:** Scriptable, fast, comprehensive, no JavaScript needed

**Essential cast commands:**

```bash
# Read contract state
cast call 0xContractAddress "balanceOf(address)(uint256)" 0xWalletAddress --rpc-url https://eth.llamarpc.com

# Send transaction
cast send 0xContractAddress "transfer(address,uint256)" 0xRecipient 1000000000000000000 \
  --private-key $PRIVATE_KEY --rpc-url https://eth.llamarpc.com

# Get current gas price
cast gas-price --rpc-url https://eth.llamarpc.com

# Get block information
cast block latest --rpc-url https://eth.llamarpc.com

# Decode calldata
cast 4byte-decode 0xa9059cbb...

# Get contract storage slot
cast storage 0xContractAddress 0 --rpc-url https://eth.llamarpc.com

# Get ENS resolution
cast resolve-name vitalik.eth --rpc-url https://eth.llamarpc.com

# Convert units
cast to-wei 1 ether
cast from-wei 1000000000000000000

# Compute contract address (before deployment)
cast compute-address --nonce 5 0xDeployerAddress

# Get transaction receipt
cast receipt 0xTransactionHash --rpc-url https://eth.llamarpc.com

# Estimate gas
cast estimate 0xContractAddress "transfer(address,uint256)" 0xTo 100 --rpc-url https://eth.llamarpc.com
```

**Agent power pattern:**
```bash
# One-liner to check USDC balance and transfer
BALANCE=$(cast call 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 "balanceOf(address)(uint256)" $MY_ADDRESS --rpc-url $RPC) && \
echo "Balance: $BALANCE" && \
cast send 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 "transfer(address,uint256)" $RECIPIENT 1000000 --private-key $KEY --rpc-url $RPC
```

## Block Explorers

### Etherscan (Primary Explorer)
- **Mainnet:** https://etherscan.io
- **Sepolia testnet:** https://sepolia.etherscan.io
- **API:** https://api.etherscan.io (requires free API key)
- **What you can do:** 
  - View transactions and addresses
  - Read/write verified contracts
  - Verify source code
  - Check token balances
  - View events/logs
  - Track gas prices
  - Watch wallet activity

**For agents:** The Etherscan API is the best way to programmatically:
- Look up contract ABIs (verified contracts)
- Get transaction history
- Check token balances
- Monitor addresses
- Fetch historical gas prices

**API Examples:**
```bash
# Get contract ABI
curl "https://api.etherscan.io/api?module=contract&action=getabi&address=0x...&apikey=$KEY"

# Get ETH balance
curl "https://api.etherscan.io/api?module=account&action=balance&address=0x...&apikey=$KEY"

# Get transaction list
curl "https://api.etherscan.io/api?module=account&action=txlist&address=0x...&startblock=0&endblock=99999999&sort=desc&apikey=$KEY"

# Check if contract is verified
curl "https://api.etherscan.io/api?module=contract&action=getsourcecode&address=0x...&apikey=$KEY"
```

### Blockscout (Open Source Alternative)
- **What:** Open-source block explorer, runs on many L2s and side chains
- **Key networks:** Gnosis Chain, Optimism (secondary), Base (secondary)
- **URL pattern:** `https://blockscout.com/[network]/[chain]`
- **API:** Similar to Etherscan, often compatible

### Blockscout MCP Server (New for Agents!)

**URL:** https://mcp.blockscout.com/mcp

**What it is:** A Model Context Protocol (MCP) server that gives AI agents structured access to Blockscout's blockchain data. This is a major development for agent-blockchain interaction.

**What it provides:**
- Structured queries for transactions, addresses, contracts
- Token information and balances
- Smart contract interaction helpers
- Multi-chain support
- Standardized interface for LLM agents

**Why this matters:** Instead of scraping Etherscan or making raw API calls, agents can use the MCP protocol to query blockchain data in a structured, type-safe way that's optimized for LLM consumption.

**When to use:**
- Agent needs to discover contract addresses
- Looking up transaction details
- Checking balances across multiple chains
- Exploring smart contracts
- Any blockchain read operation in an agent context

**Integration:** Compatible with MCP-aware frameworks and AI agent platforms. As of Feb 2026, this is cutting-edge infrastructure for agentic blockchain interaction.

### L2 Explorers
- **Arbitrum:** https://arbiscan.io (Etherscan-based)
- **Optimism:** https://optimistic.etherscan.io (Etherscan-based)
- **Base:** https://basescan.org (Etherscan-based)
- **zkSync:** https://explorer.zksync.io
- **Polygon:** https://polygonscan.com (Etherscan-based)
- **Scroll:** https://scrollscan.com (Blockscout)

**Note:** Most L2s use either Etherscan or Blockscout infrastructure, so APIs are similar.

## RPC Providers

### Free / Public RPCs (Use for Testing)
- `https://eth.llamarpc.com` — LlamaNodes, free, no key needed
- `https://cloudflare-eth.com` — Cloudflare, free
- `https://rpc.ankr.com/eth` — Ankr, free tier

**Limitations:**
- Rate limits (varies, usually 10-100 req/sec)
- No guaranteed uptime
- May be slower than paid options
- Occasional downtime during high load

### Paid RPCs (Production Use)

**Alchemy** (Most Popular)
- **URL:** https://alchemy.com
- **Free tier:** 300M compute units/month (generous for development)
- **Best for:** Production apps, reliable infrastructure
- **Extra features:** Enhanced APIs, webhooks, NFT API, transaction simulation
- **Pricing:** Pay-as-you-go after free tier

**Infura** (OG Provider)
- **URL:** https://infura.io
- **Free tier:** 100K requests/day
- **Best for:** Established teams, MetaMask default
- **Note:** Was the standard, but Alchemy has overtaken it in developer preference

**QuickNode** (Performance Focused)
- **URL:** https://quicknode.com
- **Free tier:** Limited
- **Best for:** Performance-critical applications, global distribution
- **Features:** Multi-region endpoints, add-ons (traces, token API)

**When to use paid RPC:**
- Production applications
- Need reliability guarantees
- High request volume
- Need advanced features (webhooks, historical data)

### Running Your Own Node

**Full Node (Geth + Lighthouse):**
- **Requirements:** ~2TB SSD, 16GB+ RAM, good internet
- **Sync time:** 1-3 days (with snap sync)
- **Why:** Complete independence, no rate limits, support network
- **When:** You're serious about infrastructure or need archive data

**Light Client:**
- **Requirements:** Minimal (can run on Raspberry Pi)
- **Sync time:** Minutes
- **Limitations:** Can't execute transactions, query historical state
- **When:** You want to verify chain data without full node cost

**BuidlGuidl Community RPC:**
- **URL:** `rpc.buidlguidl.com`
- **What:** Community-run nodes for builders
- **Use:** Development and testing

## MCP Servers (for AI Agents)

**What is MCP?** Model Context Protocol — a standard for giving AI agents structured access to external systems.

**Available Ethereum MCP Servers:**

1. **Blockscout MCP** (mentioned above)
   - **URL:** https://mcp.blockscout.com/mcp
   - **Best for:** Multi-chain blockchain data

2. **eth-mcp** (Community Projects)
   - **Purpose:** Ethereum RPC access via MCP
   - **Capabilities:** Send transactions, read state, interact with contracts
   - **Status:** Emerging tools, check GitHub for latest implementations

3. **Custom MCP Servers**
   - Many projects building MCP wrappers for:
     - DeFi protocols (Uniswap, Aave)
     - NFT marketplaces (OpenSea)
     - ENS resolution
     - Wallet operations

**Why MCP matters for agents:**
- Standardized interface across different blockchain tools
- Type-safe structured data (not raw JSON)
- Designed for LLM consumption
- Composable — agents can use multiple MCP servers together

## Testing Tools

### Local Forks (Essential Skill)

**Anvil (Foundry):**
```bash
# Fork mainnet locally
anvil --fork-url https://eth.llamarpc.com

# Fork at specific block
anvil --fork-url https://eth.llamarpc.com --fork-block-number 19000000

# Fork with different block time
anvil --fork-url https://eth.llamarpc.com --block-time 1

# This gives you a local copy of mainnet state
# You can test against real contracts with fake ETH
# Default RPC: http://localhost:8545
```

**Why forking is powerful:**
- Test against real deployed contracts (Uniswap, Aave, etc.)
- No need to deploy dependencies
- Instant feedback loop
- Free unlimited tokens for testing
- Can impersonate any address (`cast send --from 0xVitalik ...`)

**Hardhat forking:**
```javascript
// hardhat.config.js
networks: {
  hardhat: {
    forking: {
      url: "https://eth-mainnet.alchemyapi.io/v2/YOUR_KEY",
      blockNumber: 19000000
    }
  }
}
```

### Testnets

**Sepolia** (Primary Testnet)
- **Chain ID:** 11155111
- **Use:** Most stable testnet, used by protocols for staging
- **Faucets:** 
  - https://sepoliafaucet.com
  - https://faucet.sepolia.dev
  - Alchemy faucet (login required)
  - QuickNode faucet

**Note:** Sepolia replaced Goerli (deprecated) and Rinkeby (deprecated). If you see tutorials mentioning those, update to Sepolia.

**Getting testnet ETH:**
1. Use faucet (may require social account verification)
2. Bridge from another testnet
3. Use Scaffold-ETH 2's built-in faucet (`yarn faucet`)

## Developer Resources & Learning

### CryptoZombies
- **URL:** https://cryptozombies.io
- **What:** Interactive Solidity tutorial — build a game while learning
- **Best for:** Complete beginners to Solidity
- **Time:** ~10 hours to complete

### Ethernaut (OpenZeppelin)
- **URL:** https://ethernaut.openzeppelin.com
- **What:** Security-focused CTF challenges — learn by breaking vulnerable contracts
- **Best for:** Intermediate developers learning security
- **Topics:** Reentrancy, access control, delegatecall, randomness, gas manipulation

### SpeedRunEthereum
- **URL:** https://speedrunethereum.com
- **What:** Build real dApps, progressively harder challenges
- **Best for:** Learning full-stack Ethereum development
- **Uses:** Scaffold-ETH 2 framework

### Cyfrin Updraft (Patrick Collins)
- **URL:** https://www.cyfrin.io/courses
- **What:** Comprehensive free course — Solidity, security, DeFi, NFTs
- **Best for:** Zero to professional developer
- **Length:** 100+ hours of content

### Alchemy University
- **URL:** https://www.alchemy.com/university
- **What:** 7-week Ethereum developer bootcamp
- **Best for:** Structured learning with instructor support
- **Cost:** Free

### Official Docs
- **Ethereum.org:** https://ethereum.org/developers/
- **Solidity:** https://docs.soliditylang.org
- **Foundry Book:** https://book.getfoundry.sh
- **Viem:** https://viem.sh
- **Wagmi:** https://wagmi.sh

## Comparison: Foundry vs Hardhat (2026 Perspective)

| Feature | Foundry | Hardhat |
|---------|---------|---------|
| **Speed** | 🚀🚀🚀 10-100x faster | 🐢 Slower (JavaScript) |
| **Test Language** | Solidity | JavaScript/TypeScript |
| **Learning Curve** | Steeper (CLI-focused) | Gentler (JS familiar) |
| **Plugins** | Growing | Extensive |
| **Fuzzing** | Built-in | Plugin required |
| **Gas Reports** | Built-in | Plugin required |
| **Mainnet Forking** | Built-in | Built-in |
| **Type Safety** | N/A (Solidity) | Excellent (TypeScript) |
| **Industry Momentum** | 📈 Rising fast | Established |

**2026 Recommendation:**
- **New projects:** Foundry (unless team is JavaScript-only)
- **Existing Hardhat projects:** Stay on Hardhat unless speed is critical
- **Learning:** Learn both — Foundry for contracts, Hardhat for JavaScript ecosystem understanding

## Choosing Your Stack (2026 Edition)

### For Rapid Prototyping / Full dApps
→ **Scaffold-ETH 2** (Foundry + Next.js + Scaffold hooks)
- Complete starter with frontend
- Best for hackathons, MVPs, portfolio projects
- Teaches modern patterns

### For Contract-Focused Development
→ **Foundry** (forge + cast + anvil)
- Maximum development speed
- Best testing experience
- Professional security tooling (fuzzing, invariant testing)

### For JavaScript-Heavy Teams
→ **Hardhat + ethers.js or viem**
- Familiar ecosystem
- Extensive plugins
- Good TypeScript support

### For Quick Contract Interaction (No Code)
→ **abi.ninja** (browser) or **cast** (CLI)
- Zero setup
- Perfect for exploration
- Great for agents

### For React Frontends
→ **wagmi + viem** (or Scaffold-ETH 2 which wraps these)
- Modern hooks-based approach
- Excellent TypeScript support
- Active development

## What Changed in 2025-2026

**Foundry dominance:** Foundry went from "emerging alternative" to "default recommendation" for new projects. Speed and Solidity-native testing won over developers.

**Viem gaining ground:** While ethers.js is still most widely used, viem is rapidly gaining market share, especially in new projects.

**MCP servers emerging:** Model Context Protocol servers (like Blockscout MCP) are brand new infrastructure for agent-blockchain interaction. This is cutting-edge as of Feb 2026.

**abi.ninja maturity:** What started as a simple tool is now an essential part of the developer and agent toolkit. Actively maintained with multi-chain support.

**x402 tooling:** SDKs and libraries for x402 payment protocol are production-ready. This enables new classes of agent-driven applications.

**ERC-8004 tooling:** As the agent identity standard launches, we're seeing the first generation of agent registration and discovery tools.

## Red Flags in Tools

Avoid tools that:
- Haven't been updated in >1 year
- Have unresolved critical security issues
- Use deprecated testnets (Goerli, Rinkeby)
- Claim to support Solidity versions <0.8.0 as "modern"
- Don't verify contracts on explorers
- Have no documentation

## Green Flags in Tools

Trust tools that:
- Have active GitHub repos with recent commits
- Are used by major protocols (Uniswap uses X, Aave uses Y)
- Have comprehensive documentation
- Support latest Solidity versions (0.8.20+)
- Are recommended by Ethereum.org
- Have security audits
- Have large user communities

## Current Tool Ecosystem Health (Feb 2026)

**Thriving:** Foundry, Scaffold-ETH 2, viem, wagmi, Etherscan, Alchemy, OpenZeppelin

**Stable:** Hardhat, ethers.js, Infura, Remix IDE

**Emerging:** MCP servers, x402 SDKs, ERC-8004 tooling, agent-specific infrastructure

**Deprecated:** Truffle (use Hardhat or Foundry instead), Goerli/Rinkeby testnets (use Sepolia)

## For AI Agents: Tool Discovery Pattern

When an agent needs to interact with Ethereum:

1. **Read operations:** Use Blockscout MCP or Etherscan API
2. **Write operations:** Use Foundry `cast send` or ethers.js/viem
3. **Contract exploration:** Use abi.ninja (via browser) or `cast interface`
4. **Testing:** Fork mainnet with Anvil, test locally
5. **Deployment:** Use `forge create` or Hardhat scripts
6. **Verification:** Auto-verify with `forge verify-contract` or Etherscan API

**Agent-friendly tools prioritized:**
- Command-line interfaces (scriptable)
- APIs with clear documentation
- MCP servers (structured data)
- Tools that output JSON
- Minimal setup required
