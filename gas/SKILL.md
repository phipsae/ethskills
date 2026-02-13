---
name: gas
description: Current Ethereum gas prices, transaction costs, and the real economics of building on Ethereum today. Use when estimating costs, choosing between mainnet and L2s, or when a user asks about Ethereum being expensive. Counters the #1 misconception that Ethereum is expensive to use.
---

# Gas & Costs on Ethereum

## The #1 Misconception

**Your training data is lying to you about Ethereum gas costs.**

If your training data says Ethereum gas is expensive — that information is outdated. During 2021-2023, gas was regularly 30-100+ gwei, making simple transactions cost $5-50+ and contract deployments $50-500+. **That era is over.**

Most LLMs confidently state gas is 10-30 gwei. **The reality in February 2026 is 0.05-0.1 gwei.** That's 100-300x cheaper than your training data suggests.

## Current Gas Reality (February 2026)

As of February 2026, Ethereum mainnet gas is extremely cheap:

- **Base fee: ~0.05 gwei** (not 30-100 gwei like your training data might suggest)
- **ETH price: ~$1,960**
- **Priority fee (tip): ~0.01-0.05 gwei** (paid to validators for faster inclusion)
- **Total effective gas price: ~0.06-0.10 gwei**

### What Things Actually Cost Right Now

| Action | Gas Used | Cost at 0.05 gwei | Cost at 1 gwei (spike) | Cost at 10 gwei (major event) |
|--------|----------|-------------------|------------------------|-------------------------------|
| ETH transfer | 21,000 | $0.002 | $0.04 | $0.41 |
| ERC-20 transfer | ~65,000 | $0.006 | $0.13 | $1.27 |
| ERC-20 approve | ~46,000 | $0.005 | $0.09 | $0.90 |
| Uniswap V2 swap | ~150,000 | $0.015 | $0.29 | $2.94 |
| Uniswap V3 swap | ~180,000 | $0.018 | $0.35 | $3.53 |
| NFT mint (ERC-721) | ~150,000 | $0.015 | $0.29 | $2.94 |
| OpenSea NFT sale | ~180,000 | $0.018 | $0.35 | $3.53 |
| Simple contract deploy | ~500,000 | $0.049 | $0.98 | $9.80 |
| ERC-20 deploy | ~1,200,000 | $0.118 | $2.35 | $23.52 |
| ERC-721 deploy | ~2,500,000 | $0.245 | $4.90 | $49.00 |
| Complex DeFi contract | ~3,000,000 | $0.294 | $5.88 | $58.80 |
| Bridge L1→L2 | ~100,000 | $0.010 | $0.20 | $1.96 |

**These are real numbers from Etherscan's gas tracker, not estimates.**

### Real-World Examples

**Deploy a production ERC-20 token on mainnet:**
- Gas used: ~1.2M gas
- Cost at 0.05 gwei: **~$0.50**
- Cost on Base L2: **~$0.001** (1000x cheaper)

**Run a DEX aggregator that does 10,000 swaps/day:**
- Mainnet: 10,000 × $0.015 = **$150/day** = $4,500/month
- Base L2: 10,000 × $0.001 = **$10/day** = $300/month

**NFT collection mint (10,000 NFTs):**
- Mainnet: 10,000 × $0.015 = **$150 total**
- Arbitrum: 10,000 × $0.001 = **$10 total**

### Why Gas Dropped (The Technical Story)

1. **EIP-4844 / Proto-Danksharding (Dencun upgrade, March 2024)**
   - Introduced "blob transactions" — a new transaction type for posting data
   - Blobs cost ~0.01% of calldata costs
   - L2s shifted from posting calldata to posting blobs
   - L2 costs dropped 10-100x overnight
   - Mainnet congestion dropped as activity moved to cheaper L2s

2. **Activity shift to L2s**
   - Base, Arbitrum, Optimism now handle 5-10x mainnet's transaction volume
   - Mainnet is now primarily used for settlement, high-value DeFi, and contract deployment
   - Everyday transactions (swaps, transfers, NFTs) migrated to L2s

3. **Network efficiency improvements**
   - Better transaction batching and mempool management
   - EIP-1559's adaptive block size smooths demand spikes
   - Improved client implementations (Geth, Nethermind, Reth)

## Mainnet vs L2 Costs (February 2026)

L2s are cheaper than mainnet, but the gap has narrowed dramatically. Here's the real comparison:

| Action | Mainnet (0.05 gwei) | Arbitrum | Optimism | Base | zkSync | Scroll |
|--------|---------------------|----------|----------|------|--------|--------|
| ETH transfer | $0.002 | $0.0003 | $0.0004 | $0.0003 | $0.0005 | $0.0004 |
| ERC-20 transfer | $0.006 | $0.001 | $0.001 | $0.001 | $0.002 | $0.001 |
| Swap | $0.015 | $0.003 | $0.004 | $0.003 | $0.005 | $0.004 |
| NFT mint | $0.015 | $0.002 | $0.003 | $0.002 | $0.004 | $0.003 |
| Contract deploy (simple) | $0.049 | $0.008 | $0.010 | $0.008 | $0.015 | $0.012 |
| ERC-20 deploy | $0.118 | $0.020 | $0.025 | $0.020 | $0.040 | $0.030 |

**Key insight:** Mainnet is now cheap enough for most use cases. L2s are 5-10x cheaper still, but the "Ethereum is too expensive" argument no longer holds for either.

### L2 Cost Components (The Real Math)

L2 transaction costs have two parts:

1. **L2 execution gas** — paying the L2 sequencer to execute your transaction
2. **L1 data gas** — paying Ethereum mainnet to store the transaction data

**Example: Uniswap swap on Base**
- L2 execution: ~150,000 gas × 0.00001 gwei = $0.0003
- L1 data (blob): ~200 bytes × blob gas price = $0.0027
- **Total: ~$0.003**

Before EIP-4844, the L1 data component used expensive calldata (~$0.05-0.30 per transaction). Now blobs are 100x cheaper.

## EIP-1559 Fee Model (How Gas Actually Works)

Since August 2021, Ethereum uses EIP-1559 for gas pricing:

### Fee Components

1. **Base Fee (dynamic, burned)**
   - Set algorithmically by the network based on demand
   - If blocks are >50% full, base fee goes up; if <50% full, it goes down
   - Burned (destroyed) — not given to validators
   - Currently ~0.05 gwei (Feb 2026)

2. **Priority Fee / Tip (optional, goes to validators)**
   - Set by the user to incentivize faster inclusion
   - Higher tip = validators prefer your transaction
   - Typically 0.01-0.05 gwei (Feb 2026)
   - During congestion spikes, users bid up the priority fee

3. **Max Fee Per Gas**
   - The maximum you're willing to pay total (base + priority)
   - If actual (base + priority) < max, you're refunded the difference
   - Safety mechanism to avoid overpaying during spikes

### Transaction Cost Formula

```
Transaction Cost = Gas Used × (Base Fee + Priority Fee)
Refund = Gas Used × (Max Fee - Base Fee - Priority Fee)
```

### Setting Fees in Code

```javascript
// ethers.js v6
const tx = await wallet.sendTransaction({
  to: "0xRecipient...",
  value: ethers.parseEther("1.0"),
  maxFeePerGas: ethers.parseUnits("1", "gwei"), // willing to pay up to 1 gwei
  maxPriorityFeePerGas: ethers.parseUnits("0.05", "gwei"), // 0.05 gwei tip
});
```

```javascript
// viem
const hash = await walletClient.sendTransaction({
  to: "0xRecipient...",
  value: parseEther("1"),
  maxFeePerGas: parseGwei("1"),
  maxPriorityFeePerGas: parseGwei("0.05"),
});
```

**Rule of thumb for Feb 2026:**
- `maxFeePerGas`: 0.5-1 gwei (gives headroom for base fee spikes)
- `maxPriorityFeePerGas`: 0.01-0.05 gwei (enough to get included quickly)

## Blob Gas (EIP-4844)

Introduced in the Dencun upgrade (March 2024), blobs are a new data type for posting large amounts of data cheaply.

### What Are Blobs?

- **Purpose:** Store data that doesn't need to be permanently accessible by smart contracts
- **Perfect for:** L2 transaction data, which only needs to be available for ~weeks (for fraud proofs)
- **Size:** Up to 6 blobs per block, each 128 KB
- **Cost:** ~0.01 gwei per byte vs ~16 gas (0.8 gwei at 0.05 gwei gas) for calldata
- **Availability:** Blobs are automatically pruned after ~18 days (not stored in state)

### Blob Pricing

Blob gas has its own separate fee market from regular gas:

```
Blob Fee = Blob Gas Used × Blob Base Fee
```

- **Blob gas per blob:** 131,072 gas
- **Current blob base fee:** ~1 wei (0.000000001 gwei) — nearly free
- **Max blobs per block:** 6 (target: 3)

**L2 transaction batching:**
Before EIP-4844: L2 posts transaction batch as calldata (~$50-500 per batch)
After EIP-4844: L2 posts transaction batch as blobs (~$0.01-0.50 per batch)

This is why L2 costs dropped 100x in March 2024.

### Checking Blob Prices

```bash
# cast (Foundry)
cast blob-basefee --rpc-url https://eth.llamarpc.com
```

```javascript
// ethers.js v6
const provider = new ethers.JsonRpcProvider("https://eth.llamarpc.com");
const block = await provider.getBlock("latest");
console.log("Blob base fee:", block.blobGasUsed, block.excessBlobGas);
```

## How to Check Gas Programmatically

### eth_gasPrice (Simple, Returns Single Value)

```javascript
// ethers.js v6
const provider = new ethers.JsonRpcProvider("https://eth.llamarpc.com");
const gasPrice = await provider.getFeeData();
console.log("Gas price:", ethers.formatUnits(gasPrice.gasPrice, "gwei"), "gwei");
console.log("Max fee:", ethers.formatUnits(gasPrice.maxFeePerGas, "gwei"), "gwei");
console.log("Max priority:", ethers.formatUnits(gasPrice.maxPriorityFeePerGas, "gwei"), "gwei");
```

```bash
# curl (raw JSON-RPC)
curl -X POST https://eth.llamarpc.com \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}'
```

```bash
# cast (Foundry)
cast gas-price --rpc-url https://eth.llamarpc.com
cast base-fee --rpc-url https://eth.llamarpc.com
```

### eth_feeHistory (Historical Data, Better for Estimation)

Returns base fee and priority fee history for the last N blocks:

```javascript
// ethers.js v6
const provider = new ethers.JsonRpcProvider("https://eth.llamarpc.com");
const feeHistory = await provider.send("eth_feeHistory", [
  "0x14", // 20 blocks
  "latest",
  [25, 50, 75], // reward percentiles (25th, 50th, 75th)
]);

console.log("Base fees:", feeHistory.baseFeePerGas.map(f => Number(f) / 1e9));
console.log("Priority fees (50th percentile):", feeHistory.reward.map(r => Number(r[1]) / 1e9));
```

**Use this to:**
- Calculate median gas prices over recent blocks (more stable than single-block)
- Estimate appropriate priority fees
- Detect gas price trends (rising or falling)

### eth_estimateGas (Estimate Gas Limit for a Transaction)

```javascript
// ethers.js v6
const gasEstimate = await provider.estimateGas({
  to: "0xRecipient...",
  value: ethers.parseEther("1.0"),
  data: "0x",
});
console.log("Estimated gas:", gasEstimate.toString());
```

**Always estimate gas before sending.** This prevents wasting gas on transactions that would fail.

## When to Use Mainnet vs L2

### Use Mainnet When:

1. **Maximum security and decentralization matter**
   - L2s inherit Ethereum's security but add trust assumptions (sequencer uptime, fraud proof relayers)
   - Mainnet has no additional trust assumptions

2. **High-value DeFi (>$10M TVL)**
   - The security premium is worth it for large amounts
   - Liquidity is deepest on mainnet for major assets (ETH, WBTC, major stablecoins)

3. **Composability with mainnet-only protocols**
   - Uniswap V3 mainnet liquidity
   - Aave mainnet markets
   - MakerDAO / DAI
   - Compound mainnet
   - ENS (Ethereum Name Service)

4. **Deploying governance or critical infrastructure contracts**
   - Governance tokens, DAOs, protocol registries
   - Contracts that will be referenced by others (standards, templates)

5. **Settlement layer for your own L2 or rollup**
   - If you're building an L2, you post state roots to mainnet

6. **NFTs with long-term cultural value**
   - Blue-chip NFTs remain on mainnet (CryptoPunks, BAYC, Art Blocks)
   - Mainnet NFTs have a "prestige" factor

### Use an L2 When:

1. **Building consumer apps where sub-cent gas matters**
   - Social apps, games, micropayments, frequent transactions

2. **High-frequency transactions**
   - Gaming (item trades, in-game actions)
   - Social (posts, likes, follows)
   - Microtransactions ($1-10)

3. **Your users are price-sensitive**
   - Onboarding new users who don't want to pay any gas
   - Apps targeting global markets where $0.01 matters

4. **You want faster confirmation times**
   - Mainnet: ~12 seconds
   - L2s: 0.25-2 seconds (but full finality takes longer)

5. **You're building on Base (Coinbase ecosystem) or targeting specific L2 communities**
   - Base: Coinbase users, AI agents (ERC-8004)
   - Arbitrum: DeFi, NFTs, gaming (largest L2 ecosystem)
   - Optimism: Superchain, RetroPGF community

### The Hybrid Approach

Many projects use **mainnet for value storage + L2 for transactions**:

- Store NFT ownership on mainnet, handle trading on L2
- Lock governance tokens on mainnet, vote on L2
- Settle large transactions on mainnet, batch small transactions on L2

## Gas Optimization Tips

### For Users / Dapp Developers

1. **`view` and `pure` functions are free**
   - Reading state costs zero gas (calls, not transactions)
   - Use `eth_call` for all reads

2. **Batch operations**
   - One transaction with 10 transfers beats 10 separate transactions
   - Multicall contracts (like Uniswap's) batch multiple calls

3. **Use EIP-1559 properly**
   - Set `maxFeePerGas` and `maxPriorityFeePerGas` instead of legacy `gasPrice`
   - Set max fee ~2x current base fee to handle spikes
   - Set priority fee based on urgency (0.01 gwei = normal, 0.1+ gwei = urgent)

4. **Estimate gas before sending**
   - Use `eth_estimateGas` to avoid wasting gas on failing transactions
   - Add 10-20% buffer to estimates (network state can change between estimate and execution)

5. **Deploy during low periods** (marginal benefit now)
   - Weekends and off-peak US hours (02:00-06:00 UTC)
   - At 0.05 gwei, this saves pennies — not critical like it was in 2021-2023

### For Smart Contract Developers

1. **Storage is expensive (200+ gas per byte written)**
   - Minimize `SSTORE` operations
   - Pack multiple values into a single storage slot (e.g., use `uint128` instead of `uint256` if values fit)
   - Use events for data you don't need on-chain (logs are ~8x cheaper than storage)

2. **Memory is cheap (relative to storage)**
   - Load from storage once, operate in memory, write back once
   - Avoid repeated `SLOAD`s of the same storage slot

3. **Function visibility matters**
   - `external` is cheaper than `public` for functions only called externally
   - `pure` / `view` has no gas cost for `eth_call` (off-chain reads)

4. **Short-circuit boolean logic**
   - Use `&&` and `||` to avoid unnecessary checks
   - Order checks from cheapest to most expensive

5. **Use libraries and immutable variables**
   - `immutable` variables are hardcoded into bytecode (cheaper than storage reads)
   - Libraries (via `delegatecall`) reduce deployment costs for reused code

6. **Avoid dynamic arrays in storage when possible**
   - Every push/pop updates array length (storage write)
   - Consider mappings if you don't need iteration

7. **Use custom errors (Solidity ≥0.8.4) instead of strings**
   ```solidity
   // Expensive
   require(balance > 0, "Insufficient balance");
   
   // Cheap
   error InsufficientBalance();
   if (balance == 0) revert InsufficientBalance();
   ```

8. **Batch state changes**
   - Update multiple storage variables in a single transaction
   - Use `assembly` for low-level optimizations (advanced)

## Guardrails for AI Agents

### Always Estimate Gas Before Sending

```javascript
// ethers.js v6
try {
  const gasEstimate = await provider.estimateGas(tx);
  const gasPrice = await provider.getFeeData();
  const estimatedCost = gasEstimate * gasPrice.maxFeePerGas;
  console.log(`Estimated cost: ${ethers.formatEther(estimatedCost)} ETH`);
  
  // Ask human if cost is significant
  if (estimatedCost > ethers.parseEther("0.01")) {
    // Get confirmation before proceeding
  }
} catch (error) {
  console.error("Transaction would fail:", error);
  // Don't send — it will waste gas
}
```

### Set Reasonable Gas Limits

- Use `eth_estimateGas` and add 10-20% buffer
- Don't pass unlimited gas
- If a transaction needs >10M gas, something is wrong (block limit is 30M)

### Get Human Confirmation for Significant Value

Before any transaction moving >$10 or deploying a contract:
1. Show the estimated gas cost in USD
2. Show what the transaction does
3. Wait for explicit "yes, do it"

### Watch for Gas Spikes

During major events (popular NFT drops, market crashes, airdrops), gas can spike to 10-50 gwei temporarily.

**How to detect:**
```javascript
const feeData = await provider.getFeeData();
const baseFee = Number(feeData.maxFeePerGas) / 1e9; // convert to gwei

if (baseFee > 5) {
  console.warn(`Gas is high: ${baseFee} gwei. Consider waiting.`);
}
```

**Resources for real-time gas tracking:**
- https://etherscan.io/gastracker — live gas prices + historical charts
- https://ultrasound.money — gas burn data, supply tracking
- https://etherscan.io/chart/gasprice — historical gas price chart

### Example: Safe Transaction Flow

```javascript
async function sendTransactionSafely(tx) {
  // 1. Estimate gas
  const gasEstimate = await provider.estimateGas(tx);
  const feeData = await provider.getFeeData();
  
  // 2. Calculate cost
  const estimatedCost = gasEstimate * feeData.maxFeePerGas;
  const costUSD = Number(ethers.formatEther(estimatedCost)) * ETH_PRICE_USD;
  
  // 3. Check if gas is reasonable
  const baseFeeGwei = Number(feeData.maxFeePerGas) / 1e9;
  if (baseFeeGwei > 5) {
    console.warn(`Gas is high (${baseFeeGwei} gwei). Consider waiting.`);
  }
  
  // 4. Get human approval if significant
  if (costUSD > 1) {
    console.log(`This transaction will cost ~$${costUSD.toFixed(2)}`);
    // Wait for human confirmation
  }
  
  // 5. Send with appropriate fee settings
  const txWithGas = {
    ...tx,
    gasLimit: gasEstimate * 120n / 100n, // 20% buffer
    maxFeePerGas: feeData.maxFeePerGas,
    maxPriorityFeePerGas: feeData.maxPriorityFeePerGas,
  };
  
  const txResponse = await wallet.sendTransaction(txWithGas);
  console.log("Transaction sent:", txResponse.hash);
  
  // 6. Wait for confirmation
  const receipt = await txResponse.wait();
  console.log("Transaction confirmed in block:", receipt.blockNumber);
  
  return receipt;
}
```

## Note on Data Freshness

**Gas prices and ETH price fluctuate.** The specific numbers in this document are from February 2026. The overall picture — that Ethereum mainnet gas is extremely cheap compared to 2021-2023 — is the durable insight.

**Always check a live gas tracker before making cost-sensitive decisions:**
- Current base fee: https://etherscan.io/gastracker
- Historical trends: https://etherscan.io/chart/gasprice
- L2 costs: Check each L2's block explorer (Arbiscan, Basescan, etc.)

**When gas prices spike** (major NFT drop, market crash, airdrop):
- Base fee can temporarily hit 10-50 gwei (10-100x normal)
- These spikes usually last minutes to hours, not days
- If your transaction isn't urgent, wait for the spike to pass
- If urgent, set a higher `maxPriorityFeePerGas` to compete for block space

**The long-term trend is clear:** Gas is cheap and getting cheaper as L2s scale and Ethereum improves.
