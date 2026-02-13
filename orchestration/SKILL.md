---
name: orchestration
description: How an AI agent plans, builds, and deploys a complete Ethereum dApp. The three-phase build system for Scaffold-ETH 2 projects. Use when building a full application on Ethereum — from contracts to frontend to production deployment on IPFS.
---

# dApp Orchestration

## Overview: End-to-End dApp Development

Building an Ethereum application is a multi-layered process. This guide teaches AI agents how to approach dApp development systematically—from initial planning through production deployment.

**Core Philosophy:**
1. **Plan before code** — understand requirements, user flows, smart contract needs
2. **Iterate in phases** — contracts → frontend → production (never skip phases)
3. **Test relentlessly** — every layer, every interaction, every edge case
4. **Deploy deliberately** — localhost → testnet → production with validation at each step

---

## The Three-Phase Build System

Every Scaffold-ETH 2 project follows three phases. **Never skip phases. Never combine phases.** Each phase has its own deliverable and validation step.

### Phase Overview

| Phase | Environment | Contract Location | Frontend Location | Purpose |
|-------|-------------|-------------------|-------------------|---------|
| **Phase 1** | Local Fork | Local Hardhat/Foundry | Local Next.js | Build & iterate on contracts + UI |
| **Phase 2** | Live Network | Deployed to mainnet/L2 | Local Next.js | Test with real contracts, polish UI |
| **Phase 3** | Production | Live Network | IPFS/Vercel | Public deployment, final QA |

---

## Phase 0: Planning (Before Writing Code)

### Step 1: Define the User Journey
Write out the complete user flow in plain language:

```
Example: Token Swap dApp
1. User connects wallet
2. User selects input token (ETH) and output token (USDC)
3. User enters amount to swap
4. App calculates output amount
5. If allowance insufficient: User approves token
6. User confirms swap
7. Transaction executes
8. User sees success + updated balances
```

**Why this matters:** This becomes your testing checklist and reveals all contract interactions needed.

### Step 2: Identify Contract Requirements
From the user journey, list all contracts needed:

**Internal Contracts (you build):**
- What state needs to be stored?
- What functions will users call?
- What events need to be emitted?

**External Contracts (you integrate):**
- Token contracts (ERC20 addresses)
- Protocol contracts (Uniswap, Aave, etc.)
- Oracles (Chainlink price feeds)

### Step 3: Map Contract Interactions
Draw the interaction flow:
```
User → Frontend → Your Contract → External Protocol
                      ↓
                  Events/State Changes
                      ↓
                  Frontend Updates
```

### Step 4: Choose Tech Stack
**Scaffold-ETH 2 provides:**
- **Contracts:** Hardhat OR Foundry (pick one)
- **Frontend:** Next.js + React
- **Wallet:** RainbowKit
- **Ethereum Lib:** Wagmi + Viem

**Additional dependencies to consider:**
- OpenZeppelin (battle-tested contracts)
- Chainlink (oracles)
- External protocol SDKs (Uniswap SDK, Aave SDK)

---

## Phase 1: Scaffold (Contracts + Local Frontend)

### Phase 1.1: Smart Contracts

#### Goal
Get contracts written, compiled, tested, audited, and deployed to a local fork.

#### Setup
```bash
# Create new Scaffold-ETH 2 project
npx create-eth@latest my-dapp
cd my-dapp
yarn install

# Start local blockchain
yarn chain  # Runs Hardhat node on localhost:8545
```

#### Contract Development Steps

**1. Write Contracts**
```solidity
// packages/foundry/contracts/YourContract.sol (or packages/hardhat/contracts/)
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract TokenSwapper {
    // State variables
    address public uniswapRouter;
    
    // Constructor
    constructor(address _router) {
        uniswapRouter = _router;
    }
    
    // Main function
    function swapTokens(
        address tokenIn,
        address tokenOut,
        uint256 amountIn,
        uint256 minAmountOut
    ) external returns (uint256 amountOut) {
        // Implementation...
        emit SwapExecuted(msg.sender, tokenIn, tokenOut, amountIn, amountOut);
    }
    
    // Events
    event SwapExecuted(
        address indexed user,
        address indexed tokenIn,
        address indexed tokenOut,
        uint256 amountIn,
        uint256 amountOut
    );
}
```

**Best Practices:**
- Import from npm packages, not copy-paste
- Follow Checks-Effects-Interactions pattern
- Emit events for all state changes
- Use OpenZeppelin for standard patterns
- Add NatSpec comments

**2. Write Deploy Script**
```solidity
// packages/foundry/script/Deploy.s.sol
pragma solidity ^0.8.20;

import "forge-std/Script.sol";
import "../contracts/TokenSwapper.sol";

contract DeployScript is Script {
    function run() external {
        uint256 deployerPrivateKey = vm.envUint("DEPLOYER_PRIVATE_KEY");
        vm.startBroadcast(deployerPrivateKey);

        // Deploy with constructor args
        TokenSwapper swapper = new TokenSwapper(
            0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D // Uniswap V2 Router
        );

        console.log("TokenSwapper deployed to:", address(swapper));

        vm.stopBroadcast();
    }
}
```

**For Hardhat:**
```javascript
// packages/hardhat/deploy/00_deploy_your_contract.js
module.exports = async ({ getNamedAccounts, deployments }) => {
  const { deploy } = deployments;
  const { deployer } = await getNamedAccounts();

  await deploy("TokenSwapper", {
    from: deployer,
    args: ["0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D"],
    log: true,
  });
};
```

**3. Add External Contracts**
```typescript
// packages/nextjs/contracts/externalContracts.ts
import { GenericContractsDeclaration } from "~~/utils/scaffold-eth/contract";

const externalContracts = {
  1: {  // Mainnet
    USDC: {
      address: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
      abi: [
        "function balanceOf(address) view returns (uint256)",
        "function approve(address spender, uint256 amount) returns (bool)",
        "function allowance(address owner, address spender) view returns (uint256)",
      ],
    },
    UniswapV2Router: {
      address: "0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D",
      abi: [
        "function swapExactTokensForTokens(uint amountIn, uint amountOutMin, address[] path, address to, uint deadline) returns (uint[])",
      ],
    },
  },
} as const;

export default externalContracts;
```

**CRITICAL:** All external contracts must be added here BEFORE Phase 1.2.

**4. Deploy Locally**
```bash
# Terminal 1: Keep running
yarn chain

# Terminal 2: Deploy
yarn deploy

# Output should show:
# ✓ TokenSwapper deployed at: 0x...
# ✓ deployedContracts.ts generated
```

**5. Write Tests**
```solidity
// packages/foundry/test/TokenSwapper.t.sol
pragma solidity ^0.8.20;

import "forge-std/Test.sol";
import "../contracts/TokenSwapper.sol";

contract TokenSwapperTest is Test {
    TokenSwapper public swapper;
    address constant ROUTER = 0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D;

    function setUp() public {
        swapper = new TokenSwapper(ROUTER);
    }

    function testSwap() public {
        // Test implementation
        assertEq(swapper.uniswapRouter(), ROUTER);
    }
    
    function testSwapRevertsWithBadInput() public {
        vm.expectRevert();
        swapper.swapTokens(address(0), address(0), 0, 0);
    }
}
```

**Run tests:**
```bash
yarn foundry:test  # or yarn hardhat:test
```

**Test Coverage Goals:**
- Minimum 90% code coverage
- Test happy path
- Test all require/revert conditions
- Test edge cases (0 amounts, max uint, etc.)
- Test authorization (who can call what)
- Test reentrancy guards
- Test integration with external contracts (use fork testing)

**6. Security Audit**
Before moving to frontend, have a senior Solidity model audit for:
- Reentrancy vulnerabilities
- Integer overflow/underflow (pre-0.8.0 or with unchecked)
- Access control issues
- Front-running risks
- Oracle manipulation
- Flash loan attacks
- Gas optimization

**Fix all critical and high severity issues before Phase 1.2.**

#### Validation Checklist
- [ ] `yarn deploy` succeeds with no errors
- [ ] Contract addresses printed in console
- [ ] `deployedContracts.ts` auto-generated in `packages/nextjs/contracts/`
- [ ] Can read contract state with `cast call` or Etherscan
- [ ] All tests passing (`yarn foundry:test` or `yarn hardhat:test`)
- [ ] Test coverage ≥90%
- [ ] Security audit complete, critical issues fixed
- [ ] All external contracts added to `externalContracts.ts`

---

### Phase 1.2: Frontend (UI + Hooks + Complete UX)

#### Goal
Build the complete UI with proper UX patterns. Test the entire user journey locally with a wallet.

#### Prerequisites
- Phase 1.1 COMPLETE (contracts deployed, chain running)
- `yarn deploy` succeeded
- `deployedContracts.ts` exists

#### Setup
```bash
# Terminal 1: Keep running
yarn chain

# Terminal 2: Keep running  
yarn deploy --watch  # Auto-redeploy on contract changes

# Terminal 3: Start frontend
yarn start  # Next.js runs at http://localhost:3000
```

#### Project Structure
```
packages/nextjs/
├── app/
│   ├── page.tsx              # Home page
│   └── my-app/
│       └── page.tsx          # Your dApp page
├── components/
│   └── MyComponent.tsx       # Reusable components
├── contracts/
│   ├── deployedContracts.ts  # Auto-generated (DON'T EDIT)
│   └── externalContracts.ts  # Manual (EDIT THIS)
├── hooks/
│   └── scaffold-eth/         # Use these, not raw wagmi!
└── utils/
```

#### Scaffold-ETH 2 Hooks (Use These!)

**NEVER use raw wagmi hooks.** Always use Scaffold-ETH wrappers:

**1. Read Contract State**
```typescript
import { useScaffoldReadContract } from "~~/hooks/scaffold-eth";

// Read deployed contract
const { data: balance } = useScaffoldReadContract({
  contractName: "TokenSwapper",
  functionName: "getBalance",
  args: [address],
  watch: true,  // Auto-refresh on new blocks
});

// Read external contract
const { data: usdcBalance } = useScaffoldReadContract({
  contractName: "USDC",
  functionName: "balanceOf",
  args: [address],
});
```

**2. Write to Contract**
```typescript
import { useScaffoldWriteContract } from "~~/hooks/scaffold-eth";

const { writeContractAsync, isMining } = useScaffoldWriteContract("TokenSwapper");

// Execute transaction
const handleSwap = async () => {
  try {
    await writeContractAsync({
      functionName: "swapTokens",
      args: [tokenIn, tokenOut, amountIn, minAmountOut],
      value: parseEther("0.1"),  // If payable
      onBlockConfirmation: (txReceipt) => {
        console.log("Swap confirmed!", txReceipt);
        // Refresh UI, show success message
      },
    });
  } catch (error) {
    console.error("Swap failed:", error);
    // Show error to user
  }
};
```

**3. Get Contract Instance**
```typescript
import { useScaffoldContract } from "~~/hooks/scaffold-eth";

const { data: contract } = useScaffoldContract({
  contractName: "TokenSwapper",
  walletClient,
});

// Direct method calls
const balance = await contract?.read.getBalance([address]);
await contract?.write.swapTokens([...args]);
```

**4. Watch Events**
```typescript
import { useScaffoldEventHistory } from "~~/hooks/scaffold-eth";

const { data: swapEvents } = useScaffoldEventHistory({
  contractName: "TokenSwapper",
  eventName: "SwapExecuted",
  fromBlock: 0n,
  filters: { user: address },  // Filter by indexed params
  watch: true,
});

// Display swap history
{swapEvents?.map(event => (
  <div key={event.log.transactionHash}>
    Swapped {formatEther(event.args.amountIn)} for {formatEther(event.args.amountOut)}
  </div>
))}
```

**5. Get Network Info**
```typescript
import { useTargetNetwork } from "~~/hooks/scaffold-eth";

const { targetNetwork } = useTargetNetwork();
console.log("Building for:", targetNetwork.name);
```

#### The Three-Button Flow (MANDATORY)

Any time a user interacts with tokens, the UI must intelligently show ONE button at a time:

**1. Switch Network** (if wrong chain)  
**2. Approve Token** (if allowance insufficient)  
**3. Execute Action** (only after 1 & 2 are satisfied)

```typescript
import { useAccount, useChainId, useSwitchChain } from "wagmi";
import { useScaffoldReadContract, useScaffoldWriteContract } from "~~/hooks/scaffold-eth";
import { parseEther, formatEther } from "viem";

function SwapComponent() {
  const { address } = useAccount();
  const chainId = useChainId();
  const { switchChain } = useSwitchChain();
  const targetChainId = 1; // Mainnet

  // Read allowance
  const { data: allowance } = useScaffoldReadContract({
    contractName: "USDC",
    functionName: "allowance",
    args: [address, swapperAddress],
  });

  // Write contracts
  const { writeContractAsync: approve, isMining: isApproving } = 
    useScaffoldWriteContract("USDC");
  const { writeContractAsync: swap, isMining: isSwapping } = 
    useScaffoldWriteContract("TokenSwapper");

  const amountIn = parseEther("100"); // 100 USDC
  const needsApproval = allowance < amountIn;

  // Step 1: Check network
  if (chainId !== targetChainId) {
    return (
      <button onClick={() => switchChain({ chainId: targetChainId })}>
        Switch to Ethereum Mainnet
      </button>
    );
  }

  // Step 2: Check approval
  if (needsApproval) {
    return (
      <button 
        onClick={() => approve({
          functionName: "approve",
          args: [swapperAddress, amountIn],
        })}
        disabled={isApproving}
      >
        {isApproving ? "Approving..." : `Approve ${formatEther(amountIn)} USDC`}
      </button>
    );
  }

  // Step 3: Execute action
  return (
    <button
      onClick={() => swap({
        functionName: "swapTokens",
        args: [tokenIn, tokenOut, amountIn, minAmountOut],
      })}
      disabled={isSwapping}
    >
      {isSwapping ? "Swapping..." : "Swap Tokens"}
    </button>
  );
}
```

#### UX Rules

**1. One Button at a Time**
Never show "Approve" and "Swap" buttons simultaneously. Guide the user through the flow sequentially.

**2. Human-Readable Amounts**
```typescript
import { formatEther, formatUnits, parseEther, parseUnits } from "viem";

// Display to user
const displayAmount = formatEther(weiAmount);  // "1.5" ETH
const displayUSDC = formatUnits(usdcAmount, 6);  // "100.50" USDC

// Send to contract
const weiAmount = parseEther("1.5");  // 1500000000000000000n
const usdcAmount = parseUnits("100.50", 6);  // 100500000n
```

**3. Loading States Everywhere**
```typescript
const { data, isLoading, isError } = useScaffoldReadContract({...});

if (isLoading) return <div>Loading balance...</div>;
if (isError) return <div>Error loading balance</div>;
return <div>Balance: {formatEther(data || 0n)} ETH</div>;
```

**4. Disable Buttons During Pending Transactions**
```typescript
const { writeContractAsync, isMining } = useScaffoldWriteContract("YourContract");

<button onClick={handleSubmit} disabled={isMining || !isValid}>
  {isMining ? "Processing..." : "Submit Transaction"}
</button>
```

Blockchains take 5-12 seconds per transaction. Prevent double-submits.

**5. Helpful Error Messages**
```typescript
try {
  await writeContractAsync({...});
} catch (error) {
  if (error.message.includes("insufficient funds")) {
    setError("Not enough ETH to pay for gas");
  } else if (error.message.includes("user rejected")) {
    setError("Transaction cancelled");
  } else {
    setError("Transaction failed. Please try again.");
  }
}
```

**6. Never Use Infinite Approvals**
```typescript
// ❌ DON'T: Infinite approval
await approve({ args: [spender, maxUint256] });

// ✅ DO: Approve exact amount (or 3-5x for repeated use)
await approve({ args: [spender, amountNeeded] });
```

#### Scaffold UI Components

Use pre-built components from `~~/components/scaffold-eth/`:

```typescript
import { Address } from "~~/components/scaffold-eth";
import { AddressInput } from "~~/components/scaffold-eth";
import { Balance } from "~~/components/scaffold-eth";
import { EtherInput } from "~~/components/scaffold-eth";

// Display address with blockie and copy button
<Address address={userAddress} />

// Input with ENS resolution
<AddressInput value={recipient} onChange={setRecipient} />

// Display balance with USD toggle
<Balance address={userAddress} />

// ETH amount input with USD conversion
<EtherInput value={amount} onChange={setAmount} />
```

**See all components:** https://ui.scaffoldeth.io/

#### Testing the Full User Journey

**Critical:** Use a real wallet in the browser to test every step:

1. **Connect wallet** (MetaMask, Rainbow, etc.)
2. **Ensure you're on localhost network** (Chain ID 31337)
3. **Get test ETH** from local faucet (if using burner wallet)
4. **Walk through entire user journey** from start to finish
5. **Test error cases:**
   - Wrong network
   - Insufficient balance
   - Insufficient allowance
   - Contract reverts
   - User rejects transaction
6. **Check all UI updates:**
   - Loading states appear
   - Balances refresh after transactions
   - Events display correctly
   - Error messages are helpful

**If any step breaks, fix it before moving to Phase 2.**

#### Validation Checklist
- [ ] `yarn start` runs without errors
- [ ] Page loads and displays correctly
- [ ] Wallet connects successfully
- [ ] Can read contract state (balances, etc.)
- [ ] Three-button flow works correctly
- [ ] All transactions execute successfully
- [ ] Loading states show during pending txs
- [ ] Error messages are user-friendly
- [ ] UI updates after successful transactions
- [ ] Events display correctly
- [ ] Tested entire user journey with real wallet
- [ ] All edge cases handled gracefully

---

## Phase 2: Localhost Frontend + Production Contracts

### Goal
Deploy contracts to the live blockchain (mainnet or L2). Test the full app with real contracts, but frontend still running locally. This is where you catch integration issues and polish the UI.

### Configuration

**1. Update Target Network**
```typescript
// packages/nextjs/scaffold.config.ts
import { defineChain } from "viem";
import { mainnet, arbitrum, base, optimism } from "viem/chains";

const scaffoldConfig = {
  targetNetworks: [mainnet],  // Change from hardhat to your target chain
  // ... other config
};
```

**2. Fund Deployer Account**
```bash
# Generate new deployer (if needed)
yarn generate

# Get deployer address
yarn account

# Send ETH to deployer on target network (mainnet, Arbitrum, etc.)
# You'll need real ETH for gas!
```

**3. Set RPC URL**
```bash
# packages/hardhat/.env or packages/foundry/.env
ALCHEMY_API_KEY=your_alchemy_key
# or
DEPLOYER_PRIVATE_KEY_ENCRYPTED=your_encrypted_key
```

**4. Deploy to Live Network**
```bash
# Deploy to mainnet
yarn deploy --network mainnet

# Or L2
yarn deploy --network arbitrum
yarn deploy --network base
yarn deploy --network optimism

# Output:
# ✓ TokenSwapper deployed at: 0xYourProductionAddress
```

**5. Verify Contracts**
```bash
yarn verify --network mainnet
# Uploads source code to Etherscan for public verification
```

### Testing with Production Contracts

**1. Start Local Frontend**
```bash
yarn start
# Frontend runs at localhost:3000
# But connects to LIVE contracts!
```

**2. Test Full User Journey**
With a real wallet on the target network:
- Connect wallet
- Switch to target network (mainnet/L2)
- **IMPORTANT:** Lower test values! Use $1-10, not $1000s
- Walk through entire flow
- Verify all transactions on block explorer
- Check that contract interactions work
- Validate event emissions
- Test error handling

**3. Polish UI**

This is the time to make it look good:

**Custom Styling:**
```typescript
// packages/nextjs/tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: "#1E40AF",  // Your brand color
        secondary: "#10B981",
      },
    },
  },
};
```

**Remove Scaffold-ETH Branding:**
- Update `packages/nextjs/app/layout.tsx` (header)
- Customize footer in `packages/nextjs/components/Footer.tsx`
- Update favicon: `packages/nextjs/public/favicon.ico`
- Update metadata: title, description, OG image

**Design Principles:**
- Pick a design style that matches the dApp's purpose
- Choose 1-2 fonts and stick to them
- Use consistent spacing and sizing
- **NO LLM SLOP** — no generic purple gradients, no cookie-cutter AI design
- Make it feel unique and purposeful

**4. Lower Test Values (Optional)**
If testing is expensive, temporarily lower values in contracts:
```solidity
// Before Phase 2:
uint256 public constant MIN_STAKE = 1000 ether;

// During Phase 2 testing:
uint256 public constant MIN_STAKE = 0.01 ether;

// ⚠️ RESTORE production values before Phase 3!
```

### Validation Checklist
- [ ] Contracts deployed to target network
- [ ] Contracts verified on block explorer (Etherscan, etc.)
- [ ] `deployedContracts.ts` updated with production addresses
- [ ] Frontend connects to production contracts
- [ ] Full user journey tested with real wallet
- [ ] All transactions confirmed on block explorer
- [ ] UI polished and branded
- [ ] No Scaffold-ETH default appearance
- [ ] Test values lowered (if needed for cost)
- [ ] Ready to restore production values

---

## Phase 3: Production Deployment (IPFS/Vercel)

### Goal
Deploy the working app to IPFS or Vercel. Fully test in production environment.

### Pre-Deployment Configuration

**1. Restore Production Values**
If you lowered values in Phase 2, restore them:
```solidity
// Restore to production values
uint256 public constant MIN_STAKE = 1000 ether;
```

Redeploy if needed:
```bash
yarn deploy --network mainnet
```

**2. Configure Scaffold Config**
```typescript
// packages/nextjs/scaffold.config.ts
const scaffoldConfig = {
  targetNetworks: [mainnet],  // Or your L2
  pollingInterval: 30000,  // 30 seconds (longer than local)
  onlyLocalBurnerWallet: true,  // ⚠️ CRITICAL: Disable burner wallet!
  walletAutoConnect: false,  // Don't auto-connect
  // ... other config
};
```

**⚠️ CRITICAL: `onlyLocalBurnerWallet: true`**
This prevents the burner wallet from appearing on production. Users must connect their own wallet.

**3. Set Production RPC (Optional)**
For better performance, use your own RPC endpoint:
```typescript
// packages/nextjs/scaffold.config.ts
import { defineChain } from "viem";

const mainnetWithCustomRPC = defineChain({
  ...mainnet,
  rpcUrls: {
    default: {
      http: ["https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY"],
    },
  },
});
```

**4. Update Metadata**
```typescript
// packages/nextjs/app/layout.tsx
export const metadata = {
  title: "Your dApp Name",
  description: "Your dApp Description",
  icons: {
    icon: "/favicon.ico",
  },
  openGraph: {
    title: "Your dApp Name",
    description: "Your dApp Description",
    images: ["/og-image.png"],  // Add this!
  },
  twitter: {
    card: "summary_large_image",
    images: ["/og-image.png"],
  },
};
```

**Add OG Image:**
Create `packages/nextjs/public/og-image.png` (1200x630px)

### Deployment Methods

#### Option A: IPFS (Decentralized)

**Build & Deploy:**
```bash
# Clean previous builds
rm -rf packages/nextjs/.next packages/nextjs/out

# Build and upload to IPFS
yarn ipfs

# Output:
# ✓ Build complete
# ✓ Uploaded to IPFS
# 🌐 https://YOUR_CID.ipfs.cf-ipfs.com
# 🌐 https://ipfs.io/ipfs/YOUR_CID
```

**BuidlGuidl IPFS:**
If part of BuidlGuidl, you get a cleaner URL:
```
https://YOUR_USERNAME.buidlguidl.com
```

**IPFS Caveats:**
- First load can be slow (IPFS propagation)
- No server-side rendering
- Static export only (no API routes)

#### Option B: Vercel (Centralized, but fast)

**Setup:**
```bash
# Install Vercel CLI
npm i -g vercel

# Login
vercel login

# Deploy
cd packages/nextjs
vercel
```

**Vercel provides:**
- Fast CDN
- Server-side rendering
- API routes support
- Automatic HTTPS
- Easy custom domains

### Post-Deployment Testing

**Complete Production QA Checklist:**

**1. Basic Functionality**
- [ ] App loads on public URL
- [ ] Wallet connects correctly
- [ ] Network switching works
- [ ] Can read contract data
- [ ] Can write transactions
- [ ] Transactions confirm on block explorer

**2. User Journey Testing**
- [ ] Walk through entire user flow
- [ ] Test with multiple wallets (MetaMask, Rainbow, WalletConnect)
- [ ] Test on mobile
- [ ] Test error cases (insufficient balance, user rejection, etc.)
- [ ] Verify all UI updates happen correctly

**3. UI/UX Polish**
- [ ] No Scaffold-ETH branding leftovers
- [ ] Custom favicon loads
- [ ] Page title correct
- [ ] OG image shows in link previews (test on Twitter, Discord)
- [ ] Light and dark themes work (or theme switcher removed)
- [ ] Mobile responsive
- [ ] Loading states smooth
- [ ] Error messages helpful

**4. Performance**
- [ ] Page loads in <3 seconds
- [ ] No console errors
- [ ] Contract reads cached appropriately
- [ ] RPC calls not excessive

**5. Security**
- [ ] Burner wallet disabled (`onlyLocalBurnerWallet: true`)
- [ ] No private keys in code
- [ ] Contracts verified on block explorer
- [ ] Token approvals reasonable (not infinite)

### Common Deployment Issues

**Problem: Build fails with "Cannot find module"**
```bash
# Solution: Clean install
rm -rf node_modules packages/*/node_modules
yarn install
```

**Problem: "Wrong network" on production**
```typescript
// Solution: Check scaffold.config.ts targetNetworks matches deployed contracts
targetNetworks: [mainnet],  // Must match where contracts are deployed
```

**Problem: Burner wallet appears on production**
```typescript
// Solution: Set onlyLocalBurnerWallet
const scaffoldConfig = {
  onlyLocalBurnerWallet: true,  // ✅ This!
};
```

**Problem: RPC rate limits hit**
```typescript
// Solution: Use your own RPC endpoint
import { defineChain } from "viem";
// Add custom RPC to chain config
```

**Problem: IPFS site loads slowly**
```
// Solutions:
1. Wait 5-10 minutes for IPFS propagation
2. Use Vercel instead for faster loads
3. Pin to multiple IPFS gateways
```

### Validation Checklist
- [ ] App deployed and accessible on public URL
- [ ] Contracts deployed and verified on target network
- [ ] `onlyLocalBurnerWallet: true` set
- [ ] Full user journey works end-to-end
- [ ] All transactions verify on block explorer
- [ ] UI fully polished and branded
- [ ] OG image displays in link previews
- [ ] Mobile responsive
- [ ] No console errors
- [ ] Performance acceptable
- [ ] Security checklist passed

---

## Phase Transition Rules

### Going Backwards

**If Phase 3 reveals a bug:**
→ Go back to Phase 2 (test with localhost frontend + prod contracts)
→ Fix the issue
→ Redeploy frontend

**If Phase 2 reveals a contract bug:**
→ Go back to Phase 1 (localhost everything)
→ Fix the contract
→ Write test to prevent regression
→ Redeploy contract to production
→ Update `deployedContracts.ts`
→ Test in Phase 2 again

**Never hack around bugs in production. Fix at the right layer.**

### Never Skip Phases

**Why you can't skip Phase 1 → Phase 3:**
- No local testing = expensive mistakes on mainnet
- Can't iterate quickly
- Debugging production issues costs real gas

**Why you can't skip Phase 2:**
- Need to test with real contract interactions
- UI polish requires actual network delays
- Need to validate gas costs are reasonable

---

## AI Agent Approach to Building a dApp

### 1. Start with the End in Mind
Before writing any code:
- What problem does this solve?
- Who are the users?
- What's the complete user journey?
- What contracts are needed (internal + external)?

### 2. Plan Contract Architecture
```
User Journey → Contract Functions → State Variables → Events
```

Map out:
- What state needs to be stored on-chain?
- What can be computed off-chain?
- What external contracts will you call?
- What oracles do you need?

### 3. Build Iteratively
Don't try to build everything at once:

**Iteration 1:** Basic contract with one function
**Iteration 2:** Add error handling and events
**Iteration 3:** Add remaining functions
**Iteration 4:** Optimize gas
**Iteration 5:** Add frontend

Each iteration: code → test → validate → next

### 4. Test Everything
- Unit tests for each function
- Integration tests for contract interactions
- Fork tests against real protocols
- Frontend testing with real wallet
- Edge case testing (0, max values, reverts)
- Security testing (reentrancy, access control, etc.)

### 5. Get Feedback Early
- Deploy to testnet and share
- Ask for UI/UX feedback
- Have contracts reviewed
- Test with real users before mainnet

### 6. Document as You Go
- Comment complex logic
- Write function NatSpec
- Document user flows
- Keep README updated
- Add inline code comments

### 7. Think About Edge Cases
- What if user has 0 balance?
- What if approval is insufficient?
- What if external contract reverts?
- What if price moves mid-transaction?
- What if network is congested?

### 8. Optimize for UX
- Minimize number of transactions needed
- Batch operations where possible
- Show helpful loading states
- Provide clear error messages
- Guide users through multi-step flows

---

## Scaffold-ETH 2 Quick Reference

### Setup
```bash
# Create project
npx create-eth@latest my-dapp
cd my-dapp
yarn install

# Start local chain
yarn chain

# Deploy contracts
yarn deploy

# Watch mode (auto-redeploy on changes)
yarn deploy --watch

# Start frontend
yarn start
```

### Key Directories
```
packages/
├── foundry/ (or hardhat/)
│   ├── contracts/          # Solidity contracts
│   ├── script/             # Deploy scripts (Foundry)
│   ├── deploy/             # Deploy scripts (Hardhat)
│   └── test/               # Contract tests
└── nextjs/
    ├── app/                # Next.js pages and routes
    ├── components/         # React components
    ├── contracts/          # Contract ABIs and addresses
    │   ├── deployedContracts.ts   # Auto-generated (DON'T EDIT)
    │   └── externalContracts.ts   # Manual external contracts (EDIT THIS)
    ├── hooks/              # Custom hooks
    │   └── scaffold-eth/   # Scaffold hooks (USE THESE)
    └── scaffold.config.ts  # Main config file
```

### Configuration Files

**scaffold.config.ts** (main config):
```typescript
import { Chain } from "viem";
import { mainnet, arbitrum } from "viem/chains";

const scaffoldConfig = {
  targetNetworks: [mainnet],
  pollingInterval: 30000,
  onlyLocalBurnerWallet: true,  // Disable burner on prod
  walletAutoConnect: false,
};

export default scaffoldConfig;
```

**hardhat.config.ts** or **foundry.toml**:
Network configurations, compiler settings, etc.

### Essential Commands

**Contract Development:**
```bash
yarn compile             # Compile contracts
yarn deploy              # Deploy to configured network
yarn deploy --network mainnet
yarn verify --network mainnet
yarn test                # Run all tests
yarn foundry:test        # Foundry tests only
yarn hardhat:test        # Hardhat tests only
```

**Account Management:**
```bash
yarn generate            # Generate new deployer
yarn account             # Show deployer address & balance
yarn account:import      # Import existing private key
```

**Frontend:**
```bash
yarn start               # Start Next.js dev server
yarn build               # Production build
yarn ipfs                # Build and deploy to IPFS
```

### Scaffold Hooks API

**Read Contract:**
```typescript
const { data, isLoading, isError, refetch } = useScaffoldReadContract({
  contractName: "YourContract",
  functionName: "yourFunction",
  args: [arg1, arg2],
  watch: true,  // Auto-refresh
});
```

**Write Contract:**
```typescript
const { writeContractAsync, isMining } = useScaffoldWriteContract("YourContract");

await writeContractAsync({
  functionName: "yourFunction",
  args: [arg1, arg2],
  value: parseEther("0.1"),
  onBlockConfirmation: (receipt) => {
    console.log("Confirmed!", receipt);
  },
});
```

**Watch Events:**
```typescript
const { data: events, isLoading } = useScaffoldEventHistory({
  contractName: "YourContract",
  eventName: "YourEvent",
  fromBlock: 0n,
  watch: true,
  filters: { user: address },
});
```

**Get Contract Instance:**
```typescript
const { data: contract } = useScaffoldContract({
  contractName: "YourContract",
  walletClient,
});
```

---

## Common Patterns & Best Practices

### Token Interactions Pattern
```typescript
// 1. Check allowance
const { data: allowance } = useScaffoldReadContract({
  contractName: "TokenContract",
  functionName: "allowance",
  args: [userAddress, spenderAddress],
});

// 2. Approve if needed
if (allowance < amountNeeded) {
  await approveAsync({
    functionName: "approve",
    args: [spenderAddress, amountNeeded],
  });
}

// 3. Execute main action
await mainActionAsync({
  functionName: "mainFunction",
  args: [...],
});
```

### Multi-Step Transaction Pattern
```typescript
const [step, setStep] = useState<"approve" | "execute" | "done">("approve");

// Step 1: Approve
if (step === "approve") {
  await approve();
  setStep("execute");
}

// Step 2: Execute
if (step === "execute") {
  await execute();
  setStep("done");
}
```

### Error Handling Pattern
```typescript
try {
  await writeContractAsync({...});
} catch (error: any) {
  // Parse common errors
  if (error.message.includes("insufficient funds")) {
    showError("Not enough ETH for gas");
  } else if (error.message.includes("user rejected")) {
    showError("Transaction cancelled");
  } else if (error.message.includes("execution reverted")) {
    // Parse revert reason
    const reason = parseRevertReason(error);
    showError(`Transaction failed: ${reason}`);
  } else {
    showError("Unknown error occurred");
  }
}
```

### Loading State Pattern
```typescript
const [isLoading, setIsLoading] = useState(false);

const handleAction = async () => {
  setIsLoading(true);
  try {
    await writeContractAsync({...});
  } finally {
    setIsLoading(false);
  }
};

<button disabled={isLoading || isMining}>
  {isLoading ? "Processing..." : "Submit"}
</button>
```

---

## Resources

### Essential Reading
- **Ethereum Wingman:** https://ethwingman.com (AI agent reference)
- **Scaffold-ETH Docs:** https://docs.scaffoldeth.io/
- **Scaffold UI Components:** https://ui.scaffoldeth.io/
- **Solidity by Example:** https://solidity-by-example.org/
- **OpenZeppelin Docs:** https://docs.openzeppelin.com/

### Learning Paths
- **SpeedRunEthereum:** https://speedrunethereum.com/ (hands-on challenges)
- **ETH Tech Tree:** https://www.ethtechtree.com (advanced challenges)
- **Capture the Flag:** https://ctf.buidlguidl.com (security challenges)

### Tools
- **Viem Docs:** https://viem.sh/
- **Wagmi Docs:** https://wagmi.sh/
- **RainbowKit Docs:** https://www.rainbowkit.com/docs/introduction

---

## Troubleshooting

### "Contract not found" error
**Cause:** `deployedContracts.ts` not generated or out of sync

**Solution:**
```bash
yarn deploy  # Regenerates deployedContracts.ts
```

### "Cannot read property of undefined" on contract hook
**Cause:** Contract not deployed or wrong network

**Solution:**
- Check `yarn chain` is running (Phase 1)
- Check wallet is on correct network
- Check `scaffold.config.ts` targetNetworks

### Frontend can't find external contract
**Cause:** Not added to `externalContracts.ts`

**Solution:**
```typescript
// packages/nextjs/contracts/externalContracts.ts
const externalContracts = {
  1: {  // Chain ID
    YourExternalContract: {
      address: "0x...",
      abi: [...],
    },
  },
};
```

### Transaction fails with no error message
**Cause:** Contract revert without reason string

**Solution:**
- Check contract has proper require messages
- Use custom errors (more gas efficient)
- Check contract state in Debug Contracts UI

### IPFS build fails
**Cause:** Dynamic imports or API routes (not supported in static export)

**Solution:**
- Remove API routes
- Use static imports only
- Check next.config.js has `output: "export"`

---

This orchestration guide provides the complete framework for an AI agent to build, test, and deploy production-ready Ethereum dApps using Scaffold-ETH 2. Follow the three phases strictly, validate at each step, and never skip testing. Good luck building! 🏗️
