---
name: prediction-market
description: Binary prediction markets — pool-based vs tokenized positions, resolver mechanisms, payout math, and the edge cases LLMs miss. Use when building any kind of betting, forecasting, or outcome-resolution contract.
---

# Prediction Markets

## What You Probably Got Wrong

**You built pool-based only.** Pool-based accounting (users deposit into YES/NO pools, claim proportionally after resolution) is the simplest architecture, but it locks capital — users can't exit positions before resolution. If users will want to trade positions, you need tokenized positions (ERC-1155).

**Resolver trust is the #1 design decision, not an afterthought.** Your entire market's integrity depends on who resolves it and what happens when they don't. A single EOA resolver with no fallback means funds are locked forever if they lose their key. Decide on the resolver mechanism before writing any other code.

**You ignored one-sided markets.** If nobody bet on the winning side, there are no claimants. If nobody bet on the losing side, winners just get their stake back. Both cases cause division-by-zero or unexpected behavior if you didn't guard against them.

**You sent fees inline.** Deducting and sending fees on every claim adds gas cost and complexity. Accumulate fees and let the creator sweep them separately.

---

## Architecture Tiers

Pick the tier that matches your requirements. Don't build Tier 2 when Tier 1 is enough.

| | Tier 1 (MVP) | Tier 2 (Tradeable) | Tier 3 (Exchange) |
|---|---|---|---|
| **Trading model** | No trading — bet and claim | ERC-1155 position tokens + AMM | Position tokens + order book |
| **Pricing** | None (pool ratio visible) | Continuous via LMSR or CPMM | Limit orders, best price discovery |
| **Capital efficiency** | Low — locked until resolution | Medium — sell positions anytime | High — full price discovery |
| **Contracts** | 1 | 2-3 | 2-3 |
| **Complexity** | Low | Medium | High |

**Decision guide:** Just bet and claim → Tier 1. Trade positions before resolution with auto-pricing → Tier 2. Full exchange-grade trading → Tier 3.

**Reference:** Gnosis Conditional Tokens (`0xC59b0e4De5F1248C1140964E0fF287B192407E0C` on mainnet) is the ERC-1155 framework Polymarket is built on. If you're building Tier 2+, study their architecture: [github.com/gnosis/conditional-tokens-contracts](https://github.com/gnosis/conditional-tokens-contracts).

---

## Tier 1: Pool-Based Market (MVP)

One contract handles everything: market creation, betting, resolution, and claims.

### Contract Structure

```solidity
import {ReentrancyGuard} from "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract PredictionMarket is ReentrancyGuard {

enum Outcome { UNRESOLVED, YES, NO, CANCELLED }

struct Market {
    string question;
    address resolver;
    address creator;
    uint256 deadline;        // Betting closes after this
    uint256 resolutionDeadline; // Resolver must act by this or market can cancel
    Outcome outcome;
    uint256 yesPool;
    uint256 noPool;
    uint256 creatorFeeAccumulated;
    uint16 feeBps;           // e.g., 200 = 2%
}

mapping(uint256 => Market) public markets;
mapping(uint256 => mapping(address => uint256)) public yesBets;
mapping(uint256 => mapping(address => uint256)) public noBets;
}
```

### Key Functions

```solidity
function createMarket(
    string calldata question,
    address resolver,
    uint256 deadline,
    uint256 resolutionDeadline,
    uint16 feeBps
) external returns (uint256 marketId);

function bet(uint256 marketId, bool betYes) external payable;

function resolve(uint256 marketId, bool outcomeYes) external;
// Only resolver, only after deadline, only if UNRESOLVED

function claim(uint256 marketId) external;
// Only after resolved, proportional payout from losing pool

function cancel(uint256 marketId) external;
// Resolver or anyone after resolutionDeadline — refunds all bettors

function claimRefund(uint256 marketId) external;
// Only if CANCELLED — returns original bet

function claimCreatorFee(uint256 marketId) external;
// Creator sweeps accumulated fees
```

---

## Resolver Mechanisms

Choose based on your trust requirements:

| Mechanism | Best for | Trust | Complexity |
|-----------|----------|-------|------------|
| **Trusted EOA** | MVP, private markets | Single person | Lowest |
| **Chainlink price feed** | Numeric thresholds (price above X?) | Decentralized oracle network | Low-Medium |
| **Optimistic (UMA-style)** | Open-ended questions ("Who won the election?") | Anyone proposes, dispute period | Medium |
| **Multisig** | Higher-stakes trusted resolution | M-of-N signers | Low |

### Trusted EOA (Tier 1 Default)

Simplest: market creator designates a resolver address at creation time. The resolver calls `resolve()` after the deadline.

**Critical: always include a cancel escape hatch.** If the resolver disappears, user funds are locked forever.

**Limitation:** The resolver can bet on their own market and resolve in their favor. This is an accepted tradeoff for Tier 1 (trusted model). If this conflict of interest matters, use Chainlink or optimistic resolution instead.

```solidity
function cancel(uint256 marketId) external {
    Market storage m = markets[marketId];
    require(m.outcome == Outcome.UNRESOLVED, "Already resolved");

    if (msg.sender == m.resolver) {
        // Resolver can cancel anytime
    } else {
        // Anyone can cancel after resolution deadline passes
        require(block.timestamp > m.resolutionDeadline, "Too early");
    }

    m.outcome = Outcome.CANCELLED;
    emit MarketCancelled(marketId);
}
```

### Chainlink Price Feed Resolution

For markets with numeric thresholds ("Will ETH be above $5000 by Dec 31?"), read the price feed on-chain and resolve automatically. No human in the loop.

```solidity
import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";

struct Market {
    // ... existing fields ...
    address priceFeed;       // Chainlink feed address
    int256 strikePrice;      // Threshold to compare against
    bool resolveAbove;       // true = YES if price >= strike
}

function resolveWithPriceFeed(uint256 marketId) external {
    Market storage m = markets[marketId];
    require(block.timestamp > m.deadline, "Too early");
    require(m.outcome == Outcome.UNRESOLVED, "Already resolved");

    (, int256 price,, uint256 updatedAt,) = AggregatorV3Interface(m.priceFeed).latestRoundData();
    require(updatedAt > m.deadline, "Stale price — need post-deadline data");
    require(price > 0, "Invalid price");

    bool conditionMet = m.resolveAbove ? price >= m.strikePrice : price < m.strikePrice;
    m.outcome = conditionMet ? Outcome.YES : Outcome.NO;
}
```

**Critical:** Chainlink feeds have different decimals (ETH/USD = 8, some = 18). Match your `strikePrice` decimals to the feed's `decimals()`. See `security/SKILL.md` for staleness checks and sanity bounds.

**When NOT to use:** Open-ended questions ("Who won the election?"), subjective outcomes, anything without an on-chain data feed. Use optimistic resolution instead.

### Optimistic Resolution (UMA-style)

For open-ended questions where the answer can't be read from an on-chain feed. Anyone proposes an outcome, and if nobody disputes it within the window, it's accepted. Challengers stake a bond — loser forfeits it.

```solidity
struct Resolution {
    bool proposedOutcome;
    address proposer;
    uint256 proposedAt;
    uint256 bond;
    bool challenged;
}

uint256 constant DISPUTE_WINDOW = 24 hours;
uint256 constant MIN_BOND = 0.1 ether;
```

**Key rules:**
- Proposer must stake `MIN_BOND` when proposing
- Anyone can challenge by staking the same bond within `DISPUTE_WINDOW`
- If challenged: fall back to a trusted resolver or DAO vote to arbitrate
- If unchallenged after window: finalize the proposed outcome, return proposer's bond
- **Always have a fallback** — what happens if the dispute itself is never resolved?

---

## Pool Math & Payouts

### Proportional Payout Formula

Winners split the losing pool proportionally to their share of the winning pool:

```
payout = userBet + (userBet / winningPool) * losingPool * (1 - feeBps/10000)
```

In Solidity (multiply before divide):

```solidity
function claim(uint256 marketId) external nonReentrant {
    Market storage m = markets[marketId];
    require(m.outcome == Outcome.YES || m.outcome == Outcome.NO, "Not resolved");

    bool wonYes = m.outcome == Outcome.YES;
    uint256 userBet = wonYes ? yesBets[marketId][msg.sender] : noBets[marketId][msg.sender];
    require(userBet > 0, "No winning bet");

    uint256 winningPool = wonYes ? m.yesPool : m.noPool;
    uint256 losingPool = wonYes ? m.noPool : m.yesPool;

    // Clear bet before transfer (CEI pattern)
    if (wonYes) {
        yesBets[marketId][msg.sender] = 0;
    } else {
        noBets[marketId][msg.sender] = 0;
    }

    // Calculate bonus from losing pool (multiply before divide)
    uint256 grossBonus = losingPool > 0
        ? (userBet * losingPool) / winningPool
        : 0;

    // Fee on bonus only (not on original stake)
    uint256 fee = (grossBonus * m.feeBps) / 10_000;
    m.creatorFeeAccumulated += fee;

    uint256 payout = userBet + grossBonus - fee;

    (bool success,) = msg.sender.call{value: payout}("");
    require(success, "Transfer failed");

    emit Claimed(marketId, msg.sender, payout);
}
```

**Double-claim protection:** Zeroing the bet mapping before transfer (CEI) is sufficient — the `require(userBet > 0)` check prevents re-entry. A separate `hasClaimed` bool mapping is optional belt-and-suspenders; don't add it unless you have a specific reason.

### One-Sided Market Guards

**Nobody bet on winning side (`winningPool == 0`):** No claimants exist. The losing pool has no one to pay out to. Auto-cancel inside `resolve()` so losers can reclaim via `claimRefund()`:

```solidity
function resolve(uint256 marketId, bool outcomeYes) external {
    Market storage m = markets[marketId];
    require(m.outcome == Outcome.UNRESOLVED, "Already resolved");
    require(block.timestamp > m.deadline, "Too early");
    require(msg.sender == m.resolver, "Not resolver");

    uint256 winningPool = outcomeYes ? m.yesPool : m.noPool;
    if (winningPool == 0) {
        m.outcome = Outcome.CANCELLED;
        emit MarketCancelled(marketId);
        return;
    }
    m.outcome = outcomeYes ? Outcome.YES : Outcome.NO;
    emit MarketResolved(marketId, m.outcome);
}
```

**Nobody bet on losing side (`losingPool == 0`):** Winners get their stake back, no bonus. The `grossBonus` calculation handles this naturally (evaluates to 0).

**Guard in bet function:**
```solidity
function bet(uint256 marketId, bool betYes) external payable {
    require(msg.value > 0, "Must bet something");
    require(block.timestamp <= markets[marketId].deadline, "Betting closed");
    require(markets[marketId].outcome == Outcome.UNRESOLVED, "Already resolved");
    // ...
}
```

### Rounding Dust

The last claimer may receive slightly less due to integer division truncation. This is expected behavior, not a bug — the contract retains a few wei of dust.

**Mitigation:** Multiply before dividing. Don't try to "fix" rounding — the dust is negligible. If you want to be precise, track total claimed and let the last claimer sweep the remainder:

```solidity
// Alternative: last claimer gets whatever is left
if (totalClaimed + payout > address(this).balance) {
    payout = address(this).balance - totalFeesOwed;
}
```

### Fee Accumulation Pattern

Accumulate fees per claim, not per resolution. The creator sweeps whenever they want:

```solidity
function claimCreatorFee(uint256 marketId) external {
    Market storage m = markets[marketId];
    require(msg.sender == m.creator, "Only creator");
    uint256 fee = m.creatorFeeAccumulated;
    require(fee > 0, "No fees");

    m.creatorFeeAccumulated = 0;

    (bool success,) = msg.sender.call{value: fee}("");
    require(success, "Transfer failed");
}
```

---

## Beyond Pool-Based: Tokenized Positions (Tier 2+)

### When to Tokenize

Tokenize positions when users want to:
- **Trade before resolution** — sell a YES position if they change their mind
- **Use positions as collateral** — in lending or other DeFi protocols
- **Price discovery** — see real-time market odds via token prices

### Architecture

```
MarketFactory (creates markets)
    └── PredictionMarket.sol — market logic, resolution, settlement
    └── PositionToken.sol — ERC-1155 with YES (id=0) and NO (id=1) tokens
```

**Combined contract pattern:** For MVPs, a single contract that inherits ERC-1155 and acts as its own pool custodian is simpler than separate contracts. When the market contract holds its own ERC-1155 tokens (minting to `address(this)` for pool reserves), it must also inherit `ERC1155Holder` — otherwise `_mint(address(this), ...)` reverts with `ERC1155InvalidReceiver`:

```solidity
import {ERC1155} from "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import {ERC1155Holder} from "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Holder.sol";
import {ReentrancyGuard} from "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract PredictionMarketAMM is ERC1155, ERC1155Holder, ReentrancyGuard {
    function supportsInterface(bytes4 id) public view override(ERC1155, ERC1155Holder) returns (bool) {
        return super.supportsInterface(id);
    }
    // ...
}
```

**Size warning:** Tier 2 contracts combining ERC-1155, CPMM math, and market logic often exceed EIP-170's 24KB limit. Enable the optimizer in `foundry.toml`:

```toml
optimizer = true
optimizer_runs = 200
via_ir = true
```

The market contract mints position tokens on bet, burns them on claim. An AMM or order book provides trading.

**Key functions (Tier 2 AMM):**

```solidity
// Buy: ETH in → split to YES+NO → swap unwanted side through pool → user gets desired side
function buyYes(uint256 marketId, uint256 minYesOut) external payable;
function buyNo(uint256 marketId, uint256 minNoOut) external payable;

// Sell: user sends tokens → pool swaps to get matching pair → merge YES+NO → ETH out
function sellYes(uint256 marketId, uint256 yesAmount, uint256 minEthOut) external;
function sellNo(uint256 marketId, uint256 noAmount, uint256 minEthOut) external;
```

**`resolve()` UI note:** Always add a conditional resolve panel that shows only when `connectedAddress === market.resolver`. For test/demo apps SE2's `/debug` page is a fallback, but an in-app resolve button provides a complete user flow.

**EIP-7702 in deploy scripts:** Don't call `createMarket` in your deploy script if the creator/deployer address receives ERC-1155 tokens. On a mainnet fork, the SE2 default deployer (Anvil account #9) has EIP-7702 delegation code, causing `ERC1155InvalidReceiver` on mint. Either skip sample market creation in the deploy script, or use a fresh `vm.addr(uint256(keccak256("deployer")))` address.

### Gnosis Conditional Tokens (The Standard)

The canonical implementation. Polymarket, Augur v2, and most serious prediction markets use this framework.

**Key concepts:**
- **Conditions:** Questions with multiple outcomes
- **Positions:** ERC-1155 tokens representing bets on specific outcomes
- **Split/Merge:** Users split collateral into outcome tokens, or merge outcome tokens back into collateral
- **Redemption:** After resolution, winning outcome tokens redeem 1:1 for collateral

Contract: `0xC59b0e4De5F1248C1140964E0fF287B192407E0C` (Ethereum mainnet, deployed 2020, used by Polymarket on Polygon).

Source: [github.com/gnosis/conditional-tokens-contracts](https://github.com/gnosis/conditional-tokens-contracts)

**Don't reimplement this from scratch for Tier 2+.** Study and extend the Gnosis framework instead.

### AMM Options for Auto-Pricing

- **LMSR (Logarithmic Market Scoring Rule):** Used by Augur. Bounded loss for the market maker. Better for thin markets.
- **CPMM (Constant Product):** Uniswap-style x*y=k for outcome tokens. Simpler but unbounded loss.

Both are complex to implement correctly. For an MVP with trading, consider an off-the-shelf AMM or order book rather than building your own.

### CPMM Swap Math (Worked Example)

For a binary market with YES/NO reserves in the AMM pool:

```
Initial: yesReserve = 100, noReserve = 100, k = 10,000
User wants to buy YES tokens with 1 ETH (2% swap fee)

1. Deduct fee:    effectiveIn = 1 * 0.98 = 0.98 ETH
2. Mint NO tokens: 0.98 NO tokens added to pool (ETH backs the new tokens)
3. New NO reserve: 100 + 0.98 = 100.98
4. Apply x*y=k:   newYesReserve = k / newNoReserve = 10,000 / 100.98 = 99.03
5. YES out:        100 - 99.03 = 0.97 YES tokens sent to user

After swap: yesReserve = 99.03, noReserve = 100.98, k = 10,000
YES price moved from 0.50 → 0.505 (100.98 / (99.03 + 100.98))
Fee (0.02 ETH worth of tokens) stays in pool, increasing k over time.
```

**Key implementation detail:** The fee stays in the pool (increases k), which is the correct LP incentive — LPs earn from every trade. Don't send fees to a separate address on each swap.

### CPMM Sell Math (Reverse of Buy)

Selling is the exact reverse of buying, using the **merge** primitive:
- **Buy YES**: ETH → split to YES+NO pair → sell NO into pool (NO in → YES out via k) → user gets YES
- **Sell YES**: User gives YES to pool → pool swaps YES→NO via k → merge matched YES+NO pair → ETH out to user

```
After buy: yesReserve = 99.03, noReserve = 100.98, k ≈ 10,000
User wants to sell 0.5 YES tokens (2% fee)

1. Pool receives 0.5 YES: temp yesReserve = 99.53
2. Deduct fee:           yesInAfterFee = 0.5 * 0.98 = 0.49
3. Apply x*y=k:          newYesReserve = 99.03 + 0.49 = 99.52
4. NO out from pool:     noOut = 100.98 - (10,000 / 99.52) = 100.98 - 100.48 = 0.50
5. Merge 0.50 matched pairs: burn 0.50 YES + 0.50 NO → 0.50 ETH
6. Return 0.50 ETH to user

After sell: yesReserve = 99.52, noReserve = 100.48, k ≈ 10,000
YES price moved from 0.505 → 0.502 (selling YES pushed price down ✓)
```

**Two sell API styles:**
- **Input-specified** (simpler for MVPs): "I'm selling X tokens" → contract computes ETH out. Use `minEthOut` for slippage protection.
- **Output-specified** (Gnosis FPMM style): "I want X ETH back" → contract computes tokens needed. More UX-friendly but harder to implement.

For an MVP, use input-specified. Both are valid.

---

## Testing Patterns

### Fuzz: Payout Invariant

Total payouts must never exceed total bets (minus fees).

**Note:** Use `vm.addr(explicit_pk)` for test addresses, not `makeAddr()` — see EIP-7702 warning in `testing/SKILL.md`.

**Fuzz run count:** For financial contracts with payout math, the default 256 runs is insufficient. Set `[fuzz] runs = 1000` in `foundry.toml`.

```solidity
// Test addresses — use explicit private keys for mainnet fork compatibility (EIP-7702)
address alice = vm.addr(uint256(keccak256("alice")));
address bob = vm.addr(uint256(keccak256("bob")));
address charlie = vm.addr(uint256(keccak256("charlie")));
address resolver = vm.addr(uint256(keccak256("resolver")));
address creator = vm.addr(uint256(keccak256("creator")));
address attacker = vm.addr(uint256(keccak256("attacker")));

function testFuzz_PayoutInvariant(
    uint256 yesBet1,
    uint256 yesBet2,
    uint256 noBet1,
    bool outcomeYes  // Fuzz the resolution direction — don't hardcode YES
) public {
    yesBet1 = bound(yesBet1, 0.01 ether, 100 ether);
    yesBet2 = bound(yesBet2, 0.01 ether, 100 ether);
    noBet1 = bound(noBet1, 0.01 ether, 100 ether);

    uint256 totalBets = yesBet1 + yesBet2 + noBet1;

    // Place bets
    vm.prank(alice);
    market.bet{value: yesBet1}(marketId, true);
    vm.prank(bob);
    market.bet{value: yesBet2}(marketId, true);
    vm.prank(charlie);
    market.bet{value: noBet1}(marketId, false);

    // Resolve — fuzz both YES and NO paths
    vm.warp(deadline + 1);
    vm.prank(resolver);
    market.resolve(marketId, outcomeYes);

    // Claim winners
    if (outcomeYes) {
        vm.prank(alice);
        market.claim(marketId);
        vm.prank(bob);
        market.claim(marketId);
    } else {
        vm.prank(charlie);
        market.claim(marketId);
    }

    // The meaningful assertion: total payouts ≤ total bets
    // (assertGe(balance, 0) is trivially true — balances are unsigned)
    uint256 totalPaid = totalBets - address(market).balance;
    assertLe(totalPaid, totalBets, "Payouts exceed total bets");
}
```

### Fuzz: ETH Solvency Invariant

After all claims and fee sweeps, the contract should hold zero ETH (no funds stuck):

```solidity
function testFuzz_ContractDrainsCleanly(
    uint256 yesBet,
    uint256 noBet
) public {
    yesBet = bound(yesBet, 0.01 ether, 100 ether);
    noBet = bound(noBet, 0.01 ether, 100 ether);

    vm.prank(alice);
    market.bet{value: yesBet}(marketId, true);
    vm.prank(bob);
    market.bet{value: noBet}(marketId, false);

    vm.warp(deadline + 1);
    vm.prank(resolver);
    market.resolve(marketId, true);

    // Drain everything: winner claims, creator sweeps fees
    vm.prank(alice);
    market.claim(marketId);
    vm.prank(creator);
    market.claimCreatorFee(marketId);

    // Contract should hold only rounding dust (≤ 1 wei per claimer)
    assertLe(address(market).balance, 1, "Funds stuck in contract");
}
```

### One-Sided Market Edge Case

```solidity
function test_OneSidedMarket_NoLosingBets() public {
    // Only YES bets, no NO bets (alice/bob declared via vm.addr — see fuzz example above)
    vm.prank(alice);
    market.bet{value: 1 ether}(marketId, true);
    vm.prank(bob);
    market.bet{value: 2 ether}(marketId, true);

    vm.warp(deadline + 1);
    vm.prank(resolver);
    market.resolve(marketId, true);

    // Winners get stake back, no bonus
    uint256 aliceBefore = alice.balance;
    vm.prank(alice);
    market.claim(marketId);
    assertEq(alice.balance - aliceBefore, 1 ether);
}
```

### Resolver Abandonment (Cancel Escape Hatch)

```solidity
function test_CancelAfterResolutionDeadline() public {
    vm.warp(resolutionDeadline + 1);

    // Anyone can cancel after resolution deadline
    vm.prank(randomUser);
    market.cancel(marketId);

    // Bettors can reclaim their bets
    vm.prank(alice);
    market.claimRefund(marketId);
    assertEq(alice.balance, aliceOriginalBalance);
}
```

### Guard Function Ordering

Check state before timing — a resolved market shouldn't pass the timing check and then fail on state:

```solidity
// ✅ CORRECT — state check first, then timing
function bet(uint256 marketId, bool betYes) external payable {
    require(markets[marketId].outcome == Outcome.UNRESOLVED, "Already resolved");
    require(block.timestamp <= markets[marketId].deadline, "Betting closed");
    // ...
}
```

### Economic Assertion

Assert that attackers cannot profit, not just that the system "works":

```solidity
function test_CannotProfitFromSelfBetting() public {
    // Attacker bets both sides
    vm.startPrank(attacker);
    market.bet{value: 5 ether}(marketId, true);
    market.bet{value: 5 ether}(marketId, false);
    vm.stopPrank();

    vm.warp(deadline + 1);
    vm.prank(resolver);
    market.resolve(marketId, true);

    vm.prank(attacker);
    market.claim(marketId);

    // Attacker should not profit (loses fee on the bonus)
    assertLe(attacker.balance, attackerStartBalance, "Self-betting must not be profitable");
}
```

---

## Edge Cases Checklist

Run through this for every prediction market contract:

- [ ] **Nobody bet on winning side** — no claimants, losing pool stuck. Handle via cancellation or redistribution
- [ ] **Nobody bet on losing side** — winners get stake back, no bonus. Verify no division-by-zero
- [ ] **Resolver never resolves** — cancel/timeout fallback must exist (resolutionDeadline)
- [ ] **Betting after deadline** — must revert
- [ ] **Double-claiming** — bet mapping zeroed before transfer (CEI pattern)
- [ ] **Rounding dust on last claimer** — acceptable, but contract must not revert
- [ ] **Zero-value bet** — must revert (`require(msg.value > 0)`)
- [ ] **Resolve before deadline** — must revert (betting still open)
- [ ] **Cancel after resolution** — must revert (can't undo a resolution)
- [ ] **Fee extraction** — creator fees accumulate correctly, sweep function works
- [ ] **Self-betting (both sides)** — attacker must not profit
- [ ] **Resolver cancels while bets are open** — Trusted EOA resolver can cancel immediately after bets are placed, effectively rugging bettors. Accepted risk for Tier 1 MVP; document for users
- [ ] **Multiple markets** — state isolation between markets (no cross-contamination)

---

## Frontend: Listing Markets

**Do NOT use `useScaffoldEventHistory` with `fromBlock: 0n` on a mainnet fork.** The fork starts at block ~24M+ — scanning from block 0 returns empty results because the RPC can't handle the range. This is the #1 frontend pitfall for prediction markets and any contract that creates indexed items.

### Counter-Based Listing (Recommended)

Use a `marketCount` storage variable and index-based reads. This works reliably on any fork:

```tsx
// ✅ Counter-based listing — works on mainnet fork
const { data: marketCount } = useScaffoldReadContract({
  contractName: "PredictionMarket",
  functionName: "marketCount",
  watch: true,
});

const total = Number(marketCount ?? 0n);
const marketIds = Array.from({ length: total }, (_, i) => total - 1 - i); // newest first

return (
  <div>
    {total === 0 ? (
      <p>No markets yet</p>
    ) : (
      marketIds.map(id => <MarketCard key={id} marketId={BigInt(id)} />)
    )}
  </div>
);
```

Each `MarketCard` reads its own market data via `useScaffoldReadContract({ functionName: "markets", args: [marketId] })`.

```tsx
// ❌ Event-history listing — returns empty on mainnet fork at block 24M+
const { data: events } = useScaffoldEventHistory({
  contractName: "PredictionMarket",
  eventName: "MarketCreated",
  fromBlock: 0n, // ← This is the problem
});
```

This pattern applies to any contract that creates indexed items (auctions, vaults, staking positions, NFT listings). Always expose a counter + getter, and use counter-based iteration in the frontend.

### LP Management UI (Tier 2 AMM)

For Tier 2 AMMs, include an LP management panel alongside the trading UI. LP token holders must be able to manage their position through the app — the `/debug` page is not a substitute:

- **Add Liquidity** — ETH input + button. Mints LP tokens proportional to existing reserves.
- **Remove Liquidity** — LP token amount input + button. Burns LP tokens, returns proportional ETH.
- **Redeem LP** — Available after resolution. Burns LP tokens, returns remaining ETH (including accumulated fees).

### Slippage Protection in Frontend

The contract has `minYesOut`/`minNoOut`/`minEthOut` slippage parameters — wire them in the UI. Hardcoding `0n` disables sandwich attack protection entirely:

```tsx
// ✅ Compute 1% slippage tolerance from preview
const minOut = previewAmount * 99n / 100n;
await writeContractAsync({ functionName: "buyYes", args: [marketId, minOut], value: ethAmount });

// ❌ No protection — vulnerable to sandwich attacks
await writeContractAsync({ functionName: "buyYes", args: [marketId, 0n], value: ethAmount });
```

### Multi-Market ETH Tracking

If one contract handles multiple markets, `redeemLP` cannot use `address(this).balance` — it returns the total ETH across all markets. Track per-market liquidity separately:

```solidity
mapping(uint256 => uint256) public marketLiquidity; // ETH held per market
```

Update `marketLiquidity[marketId]` on every ETH-moving operation (buy, sell, redeem, cancel). For a single-market demo contract this isn't needed, but it's essential for production multi-market deployments.
