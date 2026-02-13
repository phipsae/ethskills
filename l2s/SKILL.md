---
name: l2s
description: Ethereum Layer 2 landscape — Arbitrum, Optimism, Base, zkSync, Scroll, Linea, and more. How they work, how to deploy on them, how to bridge, when to use which. Use when choosing an L2, deploying cross-chain, or when a user asks about Ethereum scaling.
---

# Ethereum Layer 2s

## What Are L2s?

Layer 2s (L2s) are blockchains that run on top of Ethereum (Layer 1), inheriting its security while offering cheaper and faster transactions.

**How they work:**
1. **Execution happens on the L2** — transactions are processed on the L2 chain
2. **Data is posted to Ethereum L1** — transaction data is compressed and posted to mainnet
3. **Security is inherited from L1** — Ethereum validators verify the L2's state

**Key benefits:**
- **10-100x cheaper gas** than mainnet
- **Faster transactions** (0.25-2 second blocks vs 12 seconds)
- **Same smart contract code** works on L2s (EVM-compatible)
- **Ethereum-grade security** (backed by Ethereum's consensus)

**Trade-offs:**
- **Withdrawal delays** (7 days for optimistic rollups, instant with fast bridges)
- **Additional trust assumptions** (sequencer uptime, fraud proof relayers)
- **Liquidity fragmentation** (assets spread across chains)

## The L2 Landscape (February 2026)

### Optimistic Rollups (Fraud Proof-Based)

Optimistic rollups assume transactions are valid by default. If fraud is detected, anyone can submit a fraud proof within the challenge period (~7 days).

#### **Arbitrum One**

- **Type:** Optimistic rollup (fraud proofs)
- **TVL:** $18B+ (largest L2 by TVL)
- **Stack:** Nitro (custom, WASM-based)
- **Gas cost:** ~$0.001-0.003 per transaction
- **Block time:** ~250ms (very fast)
- **Bridge:** https://bridge.arbitrum.io
- **Explorer:** https://arbiscan.io
- **RPC:** `https://arb1.arbitrum.io/rpc`
- **Chain ID:** 42161
- **Withdrawal time:** ~7 days (or instant via third-party bridges)
- **Best for:** DeFi (deepest liquidity), general purpose, NFTs, gaming

**Why choose Arbitrum:**
- Largest L2 ecosystem (most protocols deployed)
- Highest liquidity (Uniswap, GMX, Camelot, Radiant)
- Battle-tested (2+ years in production)
- Fast blocks (250ms vs 2s for most L2s)

**Key protocols on Arbitrum:**
- GMX (perps DEX, $500M+ TVL)
- Uniswap, Curve, Balancer (DEXs)
- Radiant, Aave (lending)
- Treasure DAO (gaming ecosystem)

#### **Optimism (OP Mainnet)**

- **Type:** Optimistic rollup (fraud proofs)
- **TVL:** $8B+
- **Stack:** OP Stack (open source, MIT licensed)
- **Gas cost:** ~$0.001-0.003 per transaction
- **Block time:** 2 seconds
- **Bridge:** https://app.optimism.io/bridge
- **Explorer:** https://optimistic.etherscan.io
- **RPC:** `https://mainnet.optimism.io`
- **Chain ID:** 10
- **Withdrawal time:** ~7 days (or instant via third-party bridges)
- **Best for:** General purpose, OP Stack ecosystem, public goods funding (RetroPGF)

**Why choose Optimism:**
- OP Stack is used by many L2s (Superchain vision)
- Strong community (RetroPGF, public goods funding)
- Coinbase's Base is built on OP Stack (shared ecosystem)
- Bytecode-identical to Ethereum (easiest compatibility)

**Key protocols on Optimism:**
- Velodrome (DEX, vote-escrowed model)
- Uniswap, Curve, Beethoven X
- Synthetix (perps and derivatives)
- Aave, Compound (lending)

#### **Base**

- **Type:** Optimistic rollup (fraud proofs)
- **Operated by:** Coinbase
- **TVL:** $12B+
- **Stack:** OP Stack (part of the Superchain)
- **Gas cost:** ~$0.0008-0.002 per transaction (cheapest of major L2s)
- **Block time:** 2 seconds
- **Bridge:** https://bridge.base.org
- **Explorer:** https://basescan.org
- **RPC:** `https://mainnet.base.org`
- **Chain ID:** 8453
- **Withdrawal time:** ~7 days (or instant via third-party bridges)
- **Best for:** Consumer apps, social, gaming, AI agents, onboarding from Coinbase

**Why choose Base:**
- **Easiest onboarding** — direct fiat on-ramp from Coinbase
- **Fastest-growing L2** — huge user growth in 2024-2025
- **AI agent ecosystem** — ERC-8004 (agent registry) deployed on Base
- **Consumer focus** — social apps (Farcaster), gaming, memecoins
- **Cheapest gas of major L2s** — often 50% cheaper than Arbitrum/Optimism

**Key protocols on Base:**
- Uniswap, Aerodrome (DEX)
- Aave, Moonwell (lending)
- Friend.tech, Farcaster (social)
- Blackjack, memecoins (consumer apps)

**Coinbase integration:**
- Users can withdraw to Base from Coinbase for free
- Instant on-ramp (buy crypto → deploy to Base)
- No 7-day wait for L2→Coinbase withdrawals

### ZK Rollups (Validity Proof-Based)

ZK rollups use cryptographic proofs (zero-knowledge proofs) to verify correctness. Every batch of transactions is accompanied by a proof that it was executed correctly.

**Advantages over optimistic rollups:**
- **Instant finality** — once the proof is verified on L1, it's final (no 7-day challenge period)
- **Better privacy potential** — ZK proofs can hide transaction details
- **More secure** — invalid states are impossible (proof wouldn't verify)

**Trade-offs:**
- **Slower proof generation** — complex cryptography takes time
- **Higher L1 costs** — proofs are larger than fraud proofs
- **Harder to build** — ZK circuits are complex

#### **zkSync Era**

- **Type:** ZK rollup (validity proofs)
- **TVL:** $800M+
- **Stack:** Custom zkEVM (Boojum prover)
- **Gas cost:** ~$0.003-0.008 per transaction
- **Block time:** ~1 second
- **Bridge:** https://bridge.zksync.io
- **Explorer:** https://explorer.zksync.io
- **RPC:** `https://mainnet.era.zksync.io`
- **Chain ID:** 324
- **Withdrawal time:** ~15 minutes to 1 hour (proof generation + L1 confirmation)
- **Best for:** Privacy-focused apps, ZK-native applications, ZK identity

**Gotchas:**
- **Different compilation model** — uses `zksolc` (ZK Solidity compiler)
- **Some Solidity features behave differently** — check zkSync docs before deploying
- **Deployment costs slightly higher** — ZK proof overhead

**Why choose zkSync:**
- True ZK rollup (not just EVM-compatible, ZK-native)
- Fast finality (no 7-day wait)
- Native account abstraction (all wallets are smart contracts)

#### **Scroll**

- **Type:** ZK rollup (validity proofs)
- **TVL:** $250M+
- **Stack:** zkEVM (bytecode-level compatible)
- **Gas cost:** ~$0.002-0.005 per transaction
- **Block time:** ~3 seconds
- **Bridge:** https://scroll.io/bridge
- **Explorer:** https://scrollscan.com
- **RPC:** `https://rpc.scroll.io`
- **Chain ID:** 534352
- **Withdrawal time:** ~30 minutes to 2 hours
- **Best for:** Bytecode-compatible ZK rollup — deploy Solidity without changes

**Why choose Scroll:**
- **Bytecode-compatible** — no special compiler needed (unlike zkSync)
- **True ZK rollup** with EVM equivalence
- **Fast finality** without optimistic delays
- **Growing ecosystem** (Uniswap, SyncSwap, Ambient)

#### **Linea**

- **Type:** ZK rollup (validity proofs)
- **Operated by:** Consensys (MetaMask team)
- **TVL:** $900M+
- **Stack:** zkEVM (lattice-based proofs)
- **Gas cost:** ~$0.003-0.006 per transaction
- **Block time:** ~2 seconds
- **Bridge:** https://bridge.linea.build
- **Explorer:** https://lineascan.build
- **RPC:** `https://rpc.linea.build`
- **Chain ID:** 59144
- **Withdrawal time:** ~30 minutes to 1 hour
- **Best for:** MetaMask integration, Consensys ecosystem, enterprise use cases

**Why choose Linea:**
- **MetaMask native support** — built by the MetaMask team
- **Enterprise focus** — partnerships with traditional finance
- **Fast ZK finality** without optimistic delays
- **Strong infrastructure** (Infura, Truffle, Diligence)

#### **Polygon zkEVM**

- **Type:** ZK rollup (validity proofs)
- **TVL:** $150M+
- **Stack:** zkEVM (zkProver)
- **Gas cost:** ~$0.002-0.005 per transaction
- **Block time:** ~2 seconds
- **Bridge:** https://wallet.polygon.technology/zkEVM/bridge
- **Explorer:** https://zkevm.polygonscan.com
- **RPC:** `https://zkevm-rpc.com`
- **Chain ID:** 1101
- **Withdrawal time:** ~30 minutes to 1 hour
- **Best for:** Polygon ecosystem, EVM equivalence, ZK privacy

**Note:** Polygon also has Polygon PoS (sidechain, not a true L2). zkEVM is their true L2 rollup.

### L2 Comparison Table

| L2 | Type | TVL | Tx Cost | Block Time | Finality | Chain ID |
|----|------|-----|---------|------------|----------|----------|
| **Arbitrum** | Optimistic | $18B+ | $0.001-0.003 | 250ms | 7 days | 42161 |
| **Base** | Optimistic | $12B+ | $0.0008-0.002 | 2s | 7 days | 8453 |
| **Optimism** | Optimistic | $8B+ | $0.001-0.003 | 2s | 7 days | 10 |
| **Linea** | ZK | $900M+ | $0.003-0.006 | 2s | 30-60min | 59144 |
| **zkSync Era** | ZK | $800M+ | $0.003-0.008 | 1s | 15-60min | 324 |
| **Scroll** | ZK | $250M+ | $0.002-0.005 | 3s | 30-120min | 534352 |
| **Polygon zkEVM** | ZK | $150M+ | $0.002-0.005 | 2s | 30-60min | 1101 |

**Mainnet (for comparison):** $50B+ TVL, $0.002-0.01 tx cost (0.05 gwei), 12s blocks, instant finality

## The Superchain (OP Stack Ecosystem)

The **Superchain** is Optimism's vision of many L2s built on the OP Stack sharing security, bridging, and governance.

**OP Stack L2s (as of Feb 2026):**
- **Optimism (OP Mainnet)** — the flagship
- **Base** — Coinbase's L2
- **Zora** — NFT-focused L2
- **Mode** — DeFi-focused L2
- **Public Goods Network (PGN)** — funding public goods
- **Mint** — NFT L2 by Coinbase
- **opBNB** — Binance's OP Stack L2
- **And 50+ more** (growing rapidly)

**What OP Stack enables:**
- **Shared security** — all OP Stack chains settle to Ethereum
- **Fast bridging** — native bridges between Superchain members (~1 min vs 7 days)
- **Shared governance** — RetroPGF, Optimism Collective
- **Shared sequencer** (future) — atomic transactions across Superchain

**For builders:**
- Deploy on one OP Stack chain, easy to expand to others
- Use Superchain Interop (coming 2026) for cross-chain calls
- Access to shared liquidity and infrastructure

## How to Deploy on an L2

For most L2s, deploying is **identical to mainnet** — just change the RPC URL and chain ID.

### With Foundry (forge)

```bash
# Deploy to Arbitrum
forge create src/MyContract.sol:MyContract \
  --rpc-url https://arb1.arbitrum.io/rpc \
  --private-key $PRIVATE_KEY \
  --verify --etherscan-api-key $ARBISCAN_API_KEY

# Deploy to Base
forge create src/MyContract.sol:MyContract \
  --rpc-url https://mainnet.base.org \
  --private-key $PRIVATE_KEY \
  --verify --etherscan-api-key $BASESCAN_API_KEY

# Deploy to Optimism
forge create src/MyContract.sol:MyContract \
  --rpc-url https://mainnet.optimism.io \
  --private-key $PRIVATE_KEY

# Deploy to zkSync Era (requires zksolc)
# Use zkSync's era-test-node or zkforge
```

### With Hardhat

```javascript
// hardhat.config.js
module.exports = {
  networks: {
    arbitrum: {
      url: "https://arb1.arbitrum.io/rpc",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 42161,
    },
    base: {
      url: "https://mainnet.base.org",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 8453,
    },
    optimism: {
      url: "https://mainnet.optimism.io",
      accounts: [process.env.PRIVATE_KEY],
      chainId: 10,
    },
  },
  etherscan: {
    apiKey: {
      arbitrumOne: process.env.ARBISCAN_API_KEY,
      base: process.env.BASESCAN_API_KEY,
      optimisticEthereum: process.env.OPTIMISM_API_KEY,
    },
  },
};
```

```bash
# Deploy to specific network
npx hardhat run scripts/deploy.js --network arbitrum
npx hardhat run scripts/deploy.js --network base
```

### With Scaffold-ETH 2

```typescript
// packages/hardhat/scaffold.config.ts
import { chains } from "viem/chains";

const scaffoldConfig = {
  targetNetworks: [chains.base], // or chains.arbitrum, chains.optimism
  // ... rest of config
};
```

```bash
# Deploy to configured network
yarn deploy

# Deploy to different network
yarn deploy --network arbitrum
```

### Multi-Chain Deployment (Same Address on All Chains)

Use **CREATE2** for deterministic addresses — deploy the same contract at the same address on all chains.

```solidity
// Using CREATE2
contract Factory {
  function deploy(bytes32 salt, bytes memory bytecode) public returns (address) {
    address addr;
    assembly {
      addr := create2(0, add(bytecode, 0x20), mload(bytecode), salt)
    }
    return addr;
  }
}
```

```bash
# Foundry (automatically uses CREATE2)
forge create src/MyContract.sol:MyContract \
  --rpc-url https://arb1.arbitrum.io/rpc \
  --constructor-args-path constructor-args.txt \
  --salt 0x0000000000000000000000000000000000000000000000000000000000000001

# Same command on Base, Optimism, etc. → same address!
```

**Benefits:**
- Easier to remember (one address across all chains)
- Shared branding (users know your address is the same everywhere)
- Simplified documentation

**Example:** Uniswap V3 uses the same factory address on all chains: `0x1F98431c8aD98523631AE4a59f267346ea31F984`

## Deployment Differences and Gotchas

### Optimistic Rollups (Arbitrum, Optimism, Base)

✅ **Works out of the box** — deploy like mainnet, no changes needed

**Small differences:**
- **Block numbers** increment faster (250ms-2s vs 12s)
  - Don't use `block.number` for time-based logic
  - Use `block.timestamp` instead
- **Gas price oracles** — L2s have two gas components (L2 execution + L1 data)
  - `tx.gasprice` returns effective gas price
  - `block.basefee` is the L2 base fee only

**Arbitrum-specific:**
- Uses `ArbGasInfo` precompile at `0x000000000000000000000000000000000000006C` for L1 gas estimation
- `block.number` on Arbitrum is the L1 block number (not L2)

### ZK Rollups (zkSync, Scroll, Linea)

⚠️ **zkSync Era requires special attention** — uses `zksolc` compiler

**zkSync differences:**
- **Must use `zksolc`** (ZK Solidity compiler) instead of `solc`
- **Account abstraction is native** — all accounts are smart contracts
- **`paymaster` field** in transactions (for gas sponsorship)
- **Some opcodes not supported** (e.g., `SELFDESTRUCT`, `CALLCODE`)

```bash
# Compile for zkSync Era
npm install -g @matterlabs/hardhat-zksync-solc
npx hardhat compile --network zkSyncEra
```

**Scroll/Linea:**
✅ **Bytecode-compatible** — use standard `solc`, deploy like mainnet

**What to test across L2s:**
- Block time-dependent logic (use `block.timestamp`, not `block.number`)
- Gas estimation (L2s have different gas accounting)
- Chainlink oracles (different addresses per chain)
- Cross-chain bridges (test deposits and withdrawals)

## Bridging

Moving assets between Ethereum mainnet and L2s (or between L2s).

### Official Bridges (Canonical Bridges)

Each L2 has an official bridge operated by the L2 team. These are the most secure but can be slow.

| L2 | Official Bridge | L1→L2 Time | L2→L1 Time |
|----|----------------|------------|------------|
| Arbitrum | https://bridge.arbitrum.io | ~10-15 min | ~7 days |
| Base | https://bridge.base.org | ~10-15 min | ~7 days |
| Optimism | https://app.optimism.io/bridge | ~10-15 min | ~7 days |
| zkSync Era | https://bridge.zksync.io | ~15-30 min | ~15-60 min |
| Scroll | https://scroll.io/bridge | ~15-30 min | ~30-120 min |
| Linea | https://bridge.linea.build | ~15-30 min | ~30-60 min |

**Why 7 days for optimistic rollups?**
- Challenge period for fraud proofs
- If fraud is detected, withdrawals can be blocked
- ZK rollups don't need this (proofs are verified immediately)

### Third-Party Bridges (Fast Bridges)

Third-party bridges provide instant withdrawals by fronting liquidity.

**How they work:**
1. You send tokens on L2 to the bridge contract
2. Bridge immediately releases tokens on L1 from its liquidity pool
3. Bridge claims your L2 tokens after the 7-day delay
4. You pay a small fee (~0.1-0.3%) for instant withdrawal

**Popular fast bridges:**

**Across Protocol** (https://across.to)
- Fastest bridge (30 sec - 2 min)
- Lowest fees (0.05-0.3%)
- Supports: Arbitrum, Base, Optimism, Polygon, zkSync

**Hop Protocol** (https://hop.exchange)
- Established bridge (2+ years)
- Supports: Arbitrum, Base, Optimism, Polygon, Gnosis
- 0.1-0.5% fees

**Stargate** (https://stargate.finance)
- LayerZero-based
- Supports 10+ chains
- 0.1-0.3% fees

**Synapse** (https://synapseprotocol.com)
- Multi-chain bridge
- Supports 15+ chains
- 0.1-0.5% fees

**Security warning:**
- Third-party bridges add trust assumptions
- Use official bridges for large amounts (>$100K)
- Spread risk across multiple bridges for mid-size amounts
- Check bridge TVL and audit reports

### Native Bridging (Superchain)

OP Stack chains can use native fast bridging between Superchain members:

```solidity
// Send message from Base to Optimism (Superchain Interop, coming 2026)
L2ToL2CrossDomainMessenger(0x...).sendMessage(
  optimismChainId,
  targetContract,
  message
);
```

- **~1-2 minute** transfers between Superchain L2s
- No third-party risk (native to the OP Stack)
- No additional fees (just gas)

### Cross-Chain DeFi (Advanced)

**Cross-chain swaps:**
- LiFi (https://li.fi) — aggregates bridges + DEXs for best route
- Socket (https://socket.tech) — similar meta-aggregator
- 1inch Fusion — cross-chain limit orders

**Cross-chain lending:**
- Aave Portal (discontinued) — was mainnet ↔ Polygon
- Radiant (on Arbitrum) — omnichain lending (via LayerZero)

**Cross-chain messaging:**
- LayerZero (https://layerzero.network) — generic cross-chain messaging
- Chainlink CCIP (https://chain.link/ccip) — secure cross-chain interop
- Hyperlane (https://hyperlane.xyz) — modular interop

## Choosing an L2

### Quick Decision Tree

1. **Do you need the cheapest possible gas?**
   → **Base** (~50% cheaper than Arbitrum/Optimism)

2. **Do you need the deepest DeFi liquidity?**
   → **Arbitrum** (largest TVL, most protocols)

3. **Are you building for Coinbase users?**
   → **Base** (direct on-ramp, easiest onboarding)

4. **Do you need instant finality (no 7-day wait)?**
   → **ZK rollup (zkSync, Scroll, Linea)**

5. **Are you building AI agents or social apps?**
   → **Base** (ERC-8004, Farcaster, growing consumer ecosystem)

6. **Do you want to be part of the Superchain?**
   → **Optimism or Base** (OP Stack, shared ecosystem)

7. **Do you need privacy features?**
   → **zkSync Era** (ZK-native)

8. **Do you need maximum EVM compatibility?**
   → **Scroll or Arbitrum** (bytecode-identical to Ethereum)

### L2 Selection Matrix

| Use Case | Recommended L2 | Why |
|----------|----------------|-----|
| **DeFi trading** | Arbitrum | Highest liquidity, deepest markets |
| **Consumer apps** | Base | Cheapest, fastest-growing, Coinbase on-ramp |
| **NFT marketplace** | Base or Arbitrum | Low gas, large user base |
| **Gaming** | Arbitrum or Base | Fast blocks, low cost, active ecosystems |
| **Social apps** | Base | Farcaster ecosystem, AI agents, cheap |
| **DAO governance** | Optimism | RetroPGF, governance-first culture |
| **Enterprise** | Linea | Consensys ecosystem, compliance focus |
| **Privacy** | zkSync Era | ZK-native, account abstraction |
| **Multi-chain** | Any OP Stack | Easy Superchain expansion |
| **Stable, proven** | Arbitrum | 2+ years in production, $18B TVL |

### Cost Comparison (Real Examples, Feb 2026)

| Action | Mainnet | Arbitrum | Base | Optimism | zkSync | Scroll |
|--------|---------|----------|------|----------|--------|--------|
| **ETH transfer** | $0.002 | $0.0003 | $0.0003 | $0.0004 | $0.0005 | $0.0004 |
| **Uniswap swap** | $0.015 | $0.003 | $0.002 | $0.004 | $0.005 | $0.004 |
| **NFT mint** | $0.015 | $0.002 | $0.002 | $0.003 | $0.004 | $0.003 |
| **ERC-20 deploy** | $0.118 | $0.020 | $0.018 | $0.025 | $0.040 | $0.030 |

**Example: Deploy 10 contracts + mint 10,000 NFTs**
- Mainnet: $0.118×10 + $0.015×10,000 = $151.18
- Arbitrum: $0.020×10 + $0.002×10,000 = $20.20
- **Base: $0.018×10 + $0.002×10,000 = $20.18** ← cheapest

## Multi-Chain Strategy

Many projects deploy on multiple L2s. Here's how to think about it:

### Liquidity Fragmentation

**Problem:**
- Deploy on 5 L2s → liquidity is split 5 ways
- Lower liquidity = worse prices for users
- Users on Arbitrum can't trade with users on Base

**Solutions:**
1. **Pick 1-2 primary chains** — concentrate liquidity (e.g., Arbitrum for DeFi, Base for consumers)
2. **Use cross-chain bridges** — allow users to move assets between chains
3. **Use omnichain protocols** — LayerZero, Chainlink CCIP for unified liquidity

### Deployment Strategy

**Option 1: Sequential rollout**
1. Deploy on one L2 (e.g., Base)
2. Build traction, iterate
3. Expand to second L2 (e.g., Arbitrum) when ready
4. Gradually expand to more chains

**Option 2: Multi-chain from day 1**
1. Deploy on 3-5 L2s simultaneously
2. Use same contract addresses (CREATE2)
3. Market to each L2's community separately
4. Accept liquidity fragmentation

**Option 3: Mainnet + L2s**
1. Core protocol on mainnet (security, liquidity)
2. User-facing apps on L2s (cheap transactions)
3. Bridge between them

**Recommendation for new projects:**
- Start with **1 L2** (Base or Arbitrum)
- Prove product-market fit
- Expand to 2-3 more once you have traction
- Use CREATE2 for consistent addresses

### Cross-Chain Contract Addresses (CREATE2)

Deploy at the same address on all chains:

```bash
# Step 1: Deploy on first chain with salt
forge create src/MyContract.sol:MyContract \
  --rpc-url https://mainnet.base.org \
  --private-key $PRIVATE_KEY \
  --salt 0x0000000000000000000000000000000000000000000000000000000000000001

# Output: Contract deployed at 0x1234567890abcdef...

# Step 2: Deploy on other chains with SAME salt
forge create src/MyContract.sol:MyContract \
  --rpc-url https://arb1.arbitrum.io/rpc \
  --private-key $PRIVATE_KEY \
  --salt 0x0000000000000000000000000000000000000000000000000000000000000001

# Output: Contract deployed at 0x1234567890abcdef... (SAME ADDRESS!)
```

**Requirements:**
- Same deployer address on all chains
- Same bytecode (including constructor args)
- Same salt

**Example:** You can create a factory on mainnet, then deploy from the same factory address on each L2 for consistent child addresses.

## Gas Price Tracking

Each L2 has different gas accounting:

```javascript
// ethers.js v6 - check gas on Arbitrum
const provider = new ethers.JsonRpcProvider("https://arb1.arbitrum.io/rpc");
const feeData = await provider.getFeeData();
console.log("Gas price:", ethers.formatUnits(feeData.gasPrice, "gwei"), "gwei");
```

**L2-specific gas:**
- **Optimistic rollups:** L2 execution gas + L1 data gas (blob gas after EIP-4844)
- **ZK rollups:** L2 execution gas + proof generation cost

**Arbitrum Nitro gas formula:**
```
Total Cost = L2 Gas Used × L2 Gas Price + L1 Calldata Cost
```

**Base/Optimism gas formula:**
```
Total Cost = L2 Gas Used × L2 Gas Price + (Blob Gas Used × Blob Base Fee)
```

After EIP-4844, the L1 data component dropped 100x (from calldata to blobs).

## Testing on L2s

### Use L2 Testnets

| L2 | Testnet | Chain ID | Faucet |
|----|---------|----------|--------|
| Arbitrum | Sepolia | 421614 | https://faucet.arbitrum.io |
| Base | Sepolia | 84532 | https://faucet.quicknode.com/base/sepolia |
| Optimism | Sepolia | 11155420 | https://faucet.optimism.io |
| zkSync Era | Sepolia | 300 | https://portal.zksync.io/faucet |
| Scroll | Sepolia | 534351 | https://scroll.io/faucet |

### Fork L2s Locally

```bash
# Fork Arbitrum mainnet locally
anvil --fork-url https://arb1.arbitrum.io/rpc

# Fork Base mainnet
anvil --fork-url https://mainnet.base.org

# Now deploy and test against real L2 state
```

### Hardhat Network with L2 Simulation

```javascript
// hardhat.config.js
module.exports = {
  networks: {
    hardhat: {
      forking: {
        url: "https://arb1.arbitrum.io/rpc",
        blockNumber: 180000000, // optional: pin to specific block
      },
    },
  },
};
```

```bash
# Run tests against Arbitrum fork
npx hardhat test
```

## Further Reading

- **L2Beat:** https://l2beat.com — L2 security, TVL, and risk analysis
- **Optimism Docs:** https://docs.optimism.io
- **Arbitrum Docs:** https://docs.arbitrum.io
- **Base Docs:** https://docs.base.org
- **zkSync Docs:** https://era.zksync.io/docs
- **Scroll Docs:** https://docs.scroll.io
- **Vitalik's Rollup Guide:** https://vitalik.eth.limo/general/2021/01/05/rollup.html
