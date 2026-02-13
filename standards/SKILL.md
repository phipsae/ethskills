---
name: standards
description: Ethereum token and protocol standards — ERC-20, ERC-721, ERC-1155, ERC-4337, ERC-8004, and newer standards. When to use each, how they work, key interfaces. Use when building tokens, NFTs, or choosing the right standard for a project.
---

# Ethereum Standards

## Token Standards

### ERC-20: Fungible Tokens
- **What:** The standard for fungible tokens (every token is identical). Think currencies, governance tokens, stablecoins.
- **Use when:** Building a token, stablecoin, governance token, reward token, or any fungible asset.
- **Key functions:**
  - `transfer(to, amount)` — send tokens
  - `approve(spender, amount)` — allow another address to spend your tokens
  - `transferFrom(from, to, amount)` — spend approved tokens
  - `balanceOf(address)` — check balance
  - `totalSupply()` — total tokens in existence
- **Examples:** USDC, USDT, DAI, UNI, LINK, AAVE
- **Library:** Use OpenZeppelin's `ERC20.sol` — never write your own from scratch

```solidity
// Minimal ERC-20 using OpenZeppelin
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor() ERC20("My Token", "MTK") {
        _mint(msg.sender, 1000000 * 10**decimals());
    }
}
```

### ERC-721: Non-Fungible Tokens (NFTs)
- **What:** Each token is unique with a distinct `tokenId`. Think digital art, collectibles, deeds, memberships.
- **Use when:** Each item needs to be individually identifiable and owned.
- **Key functions:**
  - `ownerOf(tokenId)` — who owns this specific token
  - `transferFrom(from, to, tokenId)` — transfer a specific token
  - `approve(to, tokenId)` — approve transfer of a specific token
  - `tokenURI(tokenId)` — metadata URI for this token
- **Examples:** Bored Ape Yacht Club, CryptoPunks (wrapped), ENS names
- **Library:** OpenZeppelin's `ERC721.sol`

### ERC-1155: Multi-Token Standard
- **What:** Supports both fungible AND non-fungible tokens in one contract. Batch operations built in.
- **Use when:** You need multiple token types in one contract (gaming items, mixed collections), or you want batch transfers.
- **Key advantage:** Single contract for all token types. One `safeTransferFrom` can move multiple different tokens.
- **Examples:** Gaming items (100 swords + 1 unique legendary sword in same contract), OpenSea collections
- **Library:** OpenZeppelin's `ERC1155.sol`

### When to Use Which

| Need | Standard |
|------|----------|
| Currency / governance token | ERC-20 |
| Unique collectibles / art | ERC-721 |
| Gaming items (mixed types) | ERC-1155 |
| Membership / access pass | ERC-721 or ERC-1155 |
| Fractionalized asset | ERC-20 (representing shares of something) |

## Identity & Account Standards

### ERC-4337: Account Abstraction
- **What:** Smart contract wallets as first-class citizens. Enables gas sponsorship, batched transactions, custom auth.
- **Status:** Live on mainnet since 2023. Growing adoption.
- **EntryPoint v0.7:** `0x0000000071727De22E5E9d8BAf0edAc6f37da032`
- **Key concepts:**
  - **UserOperation:** Like a transaction, but from a smart contract wallet
  - **Bundler:** Collects UserOperations and submits them on-chain
  - **Paymaster:** Can sponsor gas for users (gasless transactions)
- **Use when:** You want users to not need ETH for gas, or you need advanced wallet features (social recovery, session keys, spending limits)

### ERC-8004: Trustless Agents — On-Chain Agent Identity Registry

**Status:** Deployed to mainnet **January 29, 2026** — production ready with growing adoption.

**What it is:** A groundbreaking standard that establishes trust infrastructure for autonomous AI agents through three on-chain registries. This is the missing trust layer for the agentic economy.

#### Core Innovation
ERC-8004 solves a fundamental problem: how can autonomous agents operating across different organizations trust and transact with each other without pre-existing relationships? It provides portable, censorship-resistant identities that enable trustless agent-to-agent commerce.

#### Three Registry System

**1. Identity Registry (ERC-721 based)**
- Provides globally unique on-chain identities for AI agents
- Each agent is an ERC-721 NFT with a unique identifier
- Supports multiple service endpoints (A2A, MCP, OASF, ENS, DIDs, email)
- Built-in verification for agent wallets using EIP-712/ERC-1271 signatures

**Mainnet Contract Addresses:**
- **IdentityRegistry:** `0x8004A169FB4a3325136EB29fA0ceB6D2e539a432`
- **ReputationRegistry:** `0x8004BAa17C55a88189AE136b182e5fdA19dE9b63`

**Also deployed to 20+ chains:** Base, Arbitrum, Optimism, Polygon, Avalanche, Abstract, Celo, Gnosis, Linea, Mantle, MegaETH, Monad, Scroll, Taiko, BSC, and their testnets. Same addresses across chains for easy portability.

**Agent Identifier Format:**
```
agentRegistry: {namespace}:{chainId}:{identityRegistry}
Example: eip155:1:0x8004A169FB4a3325136EB29fA0ceB6D2e539a432

agentId: ERC-721 tokenId (unique per registry)
```

**2. Reputation Registry**
- Standardized feedback system with signed fixed-point values
- Tracks reputation across dimensions (uptime, success rate, quality ratings)
- Supports rich metadata (tags, endpoints, proof-of-payment)
- On-chain aggregation with off-chain scoring flexibility
- Anti-Sybil protection requires client address filtering

**Feedback Structure:**
```solidity
struct Feedback {
    int128 value;        // Signed integer rating
    uint8 valueDecimals; // 0-18 decimal places
    string tag1;         // E.g., "uptime"
    string tag2;         // E.g., "30days"
    string endpoint;     // Agent endpoint URI
    string ipfsHash;     // Optional IPFS metadata
}
```

**Example Metrics:**
- Quality rating: 87/100 → `value=87, valueDecimals=0`
- Uptime: 99.77% → `value=9977, valueDecimals=2`
- Response time: 560ms → `value=560, valueDecimals=0`

**3. Validation Registry**
- Enables independent verification of agent work
- Supports multiple trust models:
  - **Crypto-economic:** Stake-secured inference re-execution
  - **zkML:** Zero-knowledge machine learning proofs
  - **TEE:** Trusted Execution Environment oracles
- Validators respond on-chain with 0-100 scores

#### Agent Registration File

The `agentURI` (tokenURI) resolves to a JSON file with agent capabilities:

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "MyAgentName",
  "description": "Natural language description of capabilities",
  "image": "https://example.com/agent-avatar.png",
  "services": [
    {
      "name": "A2A",
      "endpoint": "https://agent.example/.well-known/agent-card.json",
      "version": "0.3.0"
    },
    {
      "name": "MCP",
      "endpoint": "https://mcp.agent.eth/",
      "version": "2025-06-18"
    }
  ],
  "x402Support": true,
  "active": true,
  "registrations": [
    {
      "agentId": 42,
      "agentRegistry": "eip155:1:0x8004A169FB4a3325136EB29fA0ceB6D2e539a432"
    }
  ],
  "supportedTrust": ["reputation", "crypto-economic", "tee-attestation"]
}
```

#### Integration Example

```solidity
// Register a new agent
uint256 agentId = identityRegistry.register(
    "ipfs://QmYourAgentRegistration",
    metadata
);

// Give feedback to agent
reputationRegistry.giveFeedback(
    agentId,
    9977,              // value: 99.77
    2,                 // valueDecimals
    "uptime",          // tag1
    "30days",          // tag2
    "https://agent.example.com/api",
    "ipfs://QmFeedbackDetails",
    keccak256(feedbackData)
);

// Get reputation summary
(uint64 count, int128 summaryValue, uint8 decimals) = 
    reputationRegistry.getSummary(
        agentId,
        trustedClientAddresses,
        "uptime",
        "30days"
    );
```

#### Use Cases

1. **Autonomous Agent Marketplaces:** Agents discover and hire other agents for specialized tasks
2. **Decentralized AI Services:** AI models expose capabilities through ERC-8004 identities
3. **Agent-to-Agent Commerce:** Combined with x402, agents autonomously pay each other
4. **Trust-Minimized Delegation:** Humans delegate tasks to agents with proven reputation
5. **Multi-Agent Coordination:** Complex workflows involving multiple specialized agents

#### Key Authors & Ecosystem Support

**Authors:** Davide Crapis (Ethereum Foundation), Marco De Rossi (MetaMask), Jordan Ellis (Google), Erik Reppel (Coinbase), Leonard Tan (MetaMask), Vitto Rivabella (EF), Isha Sangani (EF)

**Ecosystem Backing:** ENS, EigenLayer, The Graph, Taiko, major Ethereum projects

#### Resources

- **Specification:** https://eips.ethereum.org/EIPS/eip-8004
- **Contracts:** https://github.com/erc-8004/erc-8004-contracts
- **Website:** https://www.8004.org
- **Awesome List:** https://github.com/sudeepb02/awesome-erc8004

## Payment Standards

### EIP-3009: Transfer With Authorization (Gasless Transfers)

**What it is:** Allows ERC-20 token transfers via meta-transaction signatures without requiring the sender to pay gas. Used extensively by USDC and the x402 payment protocol.

**Key Innovation:** The recipient (or any third party) can submit the transaction and pay the gas, while the signature from the token holder authorizes the transfer.

**How it works:**
```solidity
function transferWithAuthorization(
    address from,
    address to,
    uint256 value,
    uint256 validAfter,
    uint256 validBefore,
    bytes32 nonce,
    uint8 v,
    bytes32 r,
    bytes32 s
) external;
```

**Use cases:**
- **x402 payments:** Server can execute payment on behalf of client
- **Gasless onboarding:** New users don't need ETH to make their first transaction
- **DeFi integrations:** Combine approval + action in single user signature
- **Agent payments:** AI agents can authorize payments without holding ETH for gas

**Adoption:** USDC on Ethereum and most chains implements EIP-3009. This is the standard that makes x402 practical for stablecoin payments.

### x402: HTTP Payment Protocol

**Status:** Production-ready open standard from Coinbase, actively deployed in 2025-2026.

**What it is:** An open protocol for internet-native payments built directly into HTTP using the long-dormant **HTTP 402 "Payment Required"** status code. Enables AI agents and humans to pay for digital services with cryptocurrency without additional API keys, accounts, or external payment flows.

**Core Innovation:** Makes payments a **first-class citizen of HTTP**, not a bolt-on. Payment negotiation happens via standard HTTP headers.

#### How x402 Works

**Standard Flow:**
```
1. Client → GET /api/data HTTP/1.1

2. Server → 402 Payment Required
   Headers:
     PAYMENT-REQUIRED: <base64-encoded payment requirements>

3. Client constructs payment, signs transaction (EIP-3009)

4. Client → GET /api/data HTTP/1.1
   Headers:
     PAYMENT-SIGNATURE: <base64-encoded payment payload>

5. Server verifies payment (locally or via facilitator)
6. Server executes/settles payment on-chain
7. Server → 200 OK
   Headers:
     PAYMENT-RESPONSE: <base64-encoded settlement response>
   Body: <requested resource>
```

#### Payment Payload Structure
```json
{
  "scheme": "exact",           // Payment scheme (exact, upto, etc.)
  "network": "eip155:8453",    // Chain identifier (Base in this example)
  "amount": "1000000",         // Amount in token smallest unit (e.g., USDC)
  "token": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "from": "0x...",             // Payer address
  "to": "0x...",               // Payee address
  "signature": "0x...",        // EIP-3009 signature
  "deadline": 1234567890,      // Expiry timestamp
  "nonce": "unique-value"      // Replay protection
}
```

#### Key Components

1. **Resource Server:** HTTP server providing an API or resource
2. **Client:** Entity (human or agent) paying for a resource
3. **Facilitator (optional):** Server that handles payment verification and blockchain submission
4. **Payment Schemes:**
   - **exact:** Transfer specific, predetermined amount (implemented)
   - **upto:** Transfer up to maximum based on resource consumption (proposed)

#### Why x402 Matters for Agents

- **No API keys:** Agents don't need to manage authentication credentials
- **Pay-per-use:** True micropayments for APIs, compute, data
- **Atomic transactions:** Payment and resource delivery are cryptographically linked
- **Network-agnostic:** Works on Ethereum, Base, Arbitrum, Optimism, Solana, etc.
- **Trust-minimizing:** Facilitators can't move funds beyond client intentions

#### x402 + ERC-8004 Synergy

The combination is powerful for autonomous agent economies:
```
1. Agent discovers service via ERC-8004 identity registry
2. Agent checks reputation via ERC-8004 reputation registry
3. Agent calls endpoint (HTTP GET)
4. Receives 402 Payment Required with x402 headers
5. Agent signs payment using EIP-3009
6. Server verifies + settles via x402 flow
7. Agent receives service, posts feedback to ERC-8004
```

This enables **fully autonomous agent economies** with trust and payments built-in.

#### Practical Example: TypeScript Client

```typescript
import { x402Fetch } from '@x402/fetch';
import { createWallet } from '@x402/evm';

const wallet = createWallet(privateKey);

const response = await x402Fetch('https://api.example.com/weather', {
  wallet,
  preferredNetwork: 'eip155:8453' // Base
});

const data = await response.json();
```

#### SDK Support

**TypeScript/JavaScript:**
```bash
npm install @x402/core @x402/evm @x402/fetch @x402/express
```

**Python:**
```bash
pip install x402
```

**Go:**
```bash
go get github.com/coinbase/x402/go
```

#### Resources

- **Official Site:** https://www.x402.org
- **GitHub:** https://github.com/coinbase/x402
- **Specification PDF:** https://www.x402.org/x402.pdf
- **Ecosystem:** https://www.x402.org/ecosystem
- **QuickNode Guide:** https://www.quicknode.com/guides/infrastructure/how-to-use-x402-payment-required

## Protocol Upgrade Standards

### EIP-7702: Set Code for EOAs (Smart Accounts)

**Status:** Deployed with **Pectra upgrade** on **May 7, 2025** — this is LIVE on mainnet.

**What it is:** The most significant UX upgrade since The Merge. Allows Externally Owned Accounts (EOAs) to temporarily delegate execution to smart contracts, bringing account abstraction benefits to existing wallets **without requiring users to migrate**.

**Key Innovation:** EOAs remain EOAs (backward compatible) but gain superpowers when needed:
- **Batch transactions:** Multiple operations in one signature
- **Gas sponsorship:** Meta-transactions / paymasters
- **Session keys:** Temporary permissions for specific actions
- **Custom authorization logic:** Arbitrary smart contract logic for approvals

**How it works:**
- User signs an authorization that sets code for their EOA during a transaction
- The EOA temporarily behaves like a smart contract for that transaction
- After transaction, EOA returns to normal
- No permanent migration required

**Impact:** Eliminates "approval fatigue" (approve then execute becomes one step), enables gasless transactions for EOA users, and brings ERC-4337-like features to regular wallets.

**Developer Note:** This is a stepping stone toward native account abstraction. Build wallet features assuming EIP-7702 support on Ethereum mainnet as of May 2025.

## Other Important Standards

### ERC-2612: Permit (Gasless Approvals)
- **What:** Allows token approvals via signature instead of a separate transaction. User signs a message, and the spender submits the approval + action in one tx.
- **Use when:** You want to save users a transaction on approval flows.
- **Note:** Works alongside EIP-3009. ERC-2612 is for approvals, EIP-3009 is for transfers.

### ERC-4626: Tokenized Vaults
- **What:** Standard for yield-bearing vaults. Deposit tokens, get share tokens back.
- **Use when:** Building vaults, yield aggregators, or staking mechanisms.
- **Examples:** Yearn vaults, ERC-4626-compatible DeFi protocols

### ERC-6551: Token Bound Accounts
- **What:** Every NFT gets its own smart contract wallet. The NFT can own assets.
- **Use when:** NFTs need to hold tokens, other NFTs, or interact with contracts.

### EIP-712: Typed Structured Data Signing
- **What:** Standard for signing structured data (not just raw bytes). Shows the user what they're signing in a readable format.
- **Use when:** Any off-chain signature scheme (permits, meta-transactions, order books, EIP-3009, x402).
- **Critical for:** User security — prevents blind signing attacks

### EIP-155: Chain Identifiers
- **What:** Standard for identifying blockchain networks
- **Format:** `eip155:{chainId}` (e.g., `eip155:1` for Ethereum mainnet, `eip155:8453` for Base)
- **Used by:** ERC-8004, x402, multi-chain protocols
- **Why it matters:** Prevents replay attacks across chains, enables clear network identification

## Finding EIP/ERC Specifications

- **Official:** https://eips.ethereum.org — all Ethereum Improvement Proposals
- **Search:** https://eips.ethereum.org/all — filterable list
- **ERC vs EIP:** ERCs (Ethereum Request for Comments) are a subset of EIPs focused on application-level standards (tokens, wallets, etc.). EIPs cover everything (consensus, networking, standards).

## What Agents Should Know About Standards

### When building with agents:
1. **ERC-8004 for identity:** Register your agent on-chain for trustless discovery
2. **x402 for payments:** Use HTTP 402 for agent-to-agent commerce
3. **EIP-3009 for transfers:** Enable gasless token transfers
4. **EIP-7702 for UX:** Build assuming smart account features on mainnet
5. **ERC-20/721/1155 for assets:** Standard interfaces ensure composability

### Security Considerations:
- **Verify contract addresses:** Always cross-reference from official sources (Etherscan, protocol docs)
- **Use standard interfaces:** Don't invent custom token standards — use ERC-20/721/1155
- **Audit integrations:** When combining standards (e.g., ERC-8004 + x402), audit the integration points
- **Test on testnets:** Deploy to Sepolia before mainnet

### Red Flags:
- Custom token interfaces that don't follow ERC standards
- Unverified contracts on Etherscan
- Standards that aren't documented on eips.ethereum.org
- Missing implementations for critical functions

### Green Flags:
- OpenZeppelin implementations
- Battle-tested contracts (Uniswap, Aave, etc.)
- Multiple audits from reputable firms
- Active community and clear documentation
- Deployed across multiple chains (e.g., ERC-8004's 20+ chains)

## Current State (February 2026)

**Production-Ready Standards:**
- ERC-20, ERC-721, ERC-1155 (battle-tested for years)
- ERC-4337 (account abstraction, growing adoption)
- EIP-7702 (smart EOAs, live since Pectra May 2025)
- ERC-8004 (agent identities, deployed Jan 29, 2026)
- EIP-3009 (gasless transfers, USDC + x402)
- x402 (HTTP payments, active deployments Q1 2026)

**These are not "coming soon" — they are LIVE and being used in production today.**
