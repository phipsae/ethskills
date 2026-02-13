---
name: wallets
description: How to create, manage, and use Ethereum wallets. Covers EOAs, smart contract wallets, multisig (Safe), and account abstraction. Essential for any AI agent that needs to interact with Ethereum — sending transactions, signing messages, or managing funds. Includes guardrails for safe key handling.
---

# Wallets on Ethereum

## Wallet Types

### EOA (Externally Owned Account)

An EOA is an Ethereum account controlled by a private key (64 hex characters / 32 bytes).

**Characteristics:**
- **Controlled by:** A private key (whoever has the key controls the account)
- **Creation cost:** Free (just generate a keypair)
- **Transaction cost:** Standard gas (21,000 gas for ETH transfer)
- **Features:** Basic — send transactions, sign messages
- **Address derivation:** `keccak256(publicKey)[12:]` (last 20 bytes of pubkey hash)

**Examples:**
- MetaMask, Rainbow, Rabby, Coinbase Wallet, Frame
- Hardware wallets (Ledger, Trezor) — EOAs with keys on secure hardware

**When to use:**
- Default choice for most users and agents
- Cheapest and simplest
- Compatible with all dApps

**Limitations:**
- Lost key = lost funds (no recovery)
- No spending limits, no multi-sig
- Can't batch transactions
- Can't have someone else pay gas

### Smart Contract Wallet

A smart contract wallet is an Ethereum account controlled by code (a deployed smart contract), not a private key.

**Characteristics:**
- **Controlled by:** Contract logic (can be arbitrarily complex)
- **Creation cost:** Gas to deploy the contract (~$0.10-0.50 on mainnet)
- **Transaction cost:** Standard gas + contract execution overhead (10-30% more)
- **Features:** Programmable — multisig, spending limits, recovery, session keys, gas sponsorship
- **Address:** The contract's deployed address

**Examples:**
- Safe (formerly Gnosis Safe) — multisig
- Argent — social recovery
- Kernel, Biconomy — account abstraction (ERC-4337)

**When to use:**
- Need multisig (multiple people control funds)
- Need recovery mechanisms (lost keys don't mean lost funds)
- Need advanced features (spending limits, session keys, batching)
- Building consumer apps (sponsor gas for users)

**Limitations:**
- Costs gas to deploy
- Slightly more expensive per transaction
- Not all dApps support smart contract wallets (some check `msg.sender` is an EOA)

### EIP-7702: Smart EOAs (Pectra Upgrade, May 2025)

**This is the future.** EIP-7702, included in the Pectra upgrade (May 2025), allows EOAs to **temporarily delegate control to a smart contract** within a single transaction.

**How it works:**
1. EOA signs an "authorization" to delegate to a contract
2. During transaction execution, the EOA's code becomes the contract's code
3. The contract can execute complex logic (batching, gas sponsorship, etc.)
4. After the transaction, the EOA returns to normal

**Why this matters:**
- **Best of both worlds:** EOAs get smart contract features without deploying a new wallet
- **Backwards compatible:** Your existing EOA address keeps working
- **Seamless UX:** Users don't need to migrate wallets or addresses
- **Gas efficient:** No permanent contract deployment needed

**Example use case:**
An agent with an EOA wallet wants to batch 10 token approvals into one transaction:

```solidity
// Authorization (signed by EOA)
Authorization {
  chainId: 1,
  address: 0xBatchExecutor... (contract to delegate to),
  nonce: 0
}

// Transaction with EIP-7702 authorization
{
  to: EOA_ADDRESS,
  data: batchCalldata,
  authorizationList: [signedAuthorization]
}
```

The EOA temporarily becomes the BatchExecutor contract, executes the batch, then returns to normal.

**Status (Feb 2026):** Deployed on mainnet (Pectra, May 2025). Wallet support growing (MetaMask, Rainbow adding support). Still early — use standard EOAs or smart contract wallets for production agents until tooling matures.

### Account Abstraction (ERC-4337)

ERC-4337 is a standard for smart contract wallets that enables advanced features without protocol changes.

**Key Components:**

1. **UserOperation:** Like a transaction, but signed by a smart contract wallet
2. **Bundler:** A node that collects UserOperations and submits them to the EntryPoint contract
3. **EntryPoint:** A singleton contract that executes UserOperations (address: `0x0000000071727De22E5E9d8BAf0edAc6f37da032` on mainnet)
4. **Paymaster:** Optional contract that sponsors gas for users (pays gas on their behalf)

**What ERC-4337 Enables:**

- **Gas sponsorship:** Apps can pay gas for their users
- **Batched transactions:** Execute multiple operations atomically
- **Custom signature schemes:** Use passkeys, biometrics, social recovery instead of private keys
- **Session keys:** Temporary keys with limited permissions (e.g., "can spend up to 1 ETH on Uniswap")

**Example Flow:**

1. User creates a UserOperation: "Swap 1 ETH for USDC on Uniswap"
2. Bundler submits it to the EntryPoint contract
3. EntryPoint validates the signature (via the wallet contract's validation logic)
4. If valid, EntryPoint executes the call
5. Paymaster (if configured) pays the gas

**Status (Feb 2026):** Growing adoption but still early. Major implementations:
- Kernel (by ZeroDev)
- Biconomy
- Alchemy's Account Kit
- Pimlico infrastructure

**For AI agents:** ERC-4337 is powerful but adds complexity. Use standard EOAs or Safe multisigs unless you specifically need account abstraction features.

## Creating a Wallet for an AI Agent

### Simple EOA (Recommended Starting Point)

#### Using ethers.js v6

```javascript
import { Wallet, JsonRpcProvider } from "ethers";

// Generate a new random wallet
const wallet = Wallet.createRandom();
console.log("Address:", wallet.address);
console.log("Private key:", wallet.privateKey);
// ⚠️ NEVER log private keys in production. This is for illustration only.

// Derive from mnemonic (12 or 24 words)
const mnemonic = "word1 word2 word3 ...";
const walletFromMnemonic = Wallet.fromPhrase(mnemonic);

// Import existing private key
const existingWallet = new Wallet("0x1234567890abcdef...");

// Connect to a provider to send transactions
const provider = new JsonRpcProvider("https://eth.llamarpc.com");
const connectedWallet = wallet.connect(provider);

// Now you can send transactions
const tx = await connectedWallet.sendTransaction({
  to: "0xRecipient...",
  value: ethers.parseEther("1.0"),
});
console.log("Transaction hash:", tx.hash);
await tx.wait(); // Wait for confirmation
console.log("Transaction confirmed!");
```

#### Using viem

```javascript
import { createWalletClient, http, parseEther } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { mainnet } from "viem/chains";

// Generate a new wallet
import { generatePrivateKey } from "viem/accounts";
const privateKey = generatePrivateKey();
const account = privateKeyToAccount(privateKey);
console.log("Address:", account.address);

// Import existing private key
const account = privateKeyToAccount("0x1234567890abcdef...");

// Create wallet client
const walletClient = createWalletClient({
  account,
  chain: mainnet,
  transport: http("https://eth.llamarpc.com"),
});

// Send transaction
const hash = await walletClient.sendTransaction({
  to: "0xRecipient...",
  value: parseEther("1"),
});
console.log("Transaction hash:", hash);
```

#### Using Foundry's cast

```bash
# Generate a new wallet
cast wallet new

# Output:
# Successfully created new keypair.
# Address: 0x1234567890abcdefabcdefabcdefabcdefabcdef
# Private key: 0x...

# Generate with mnemonic
cast wallet new --json | jq -r .mnemonic

# Derive address from private key
cast wallet address --private-key 0x...

# Sign a message
cast wallet sign "Hello, Ethereum!" --private-key 0x...
```

### Signing Messages

Wallets can sign arbitrary data (not just transactions). This is used for authentication and off-chain verification.

```javascript
// ethers.js v6
const message = "Login to MyDapp";
const signature = await wallet.signMessage(message);
console.log("Signature:", signature);

// Verify signature
const recoveredAddress = ethers.verifyMessage(message, signature);
console.log("Signer:", recoveredAddress);
console.log("Match:", recoveredAddress === wallet.address);
```

```javascript
// viem
import { signMessage, verifyMessage } from "viem/accounts";

const signature = await walletClient.signMessage({
  message: "Login to MyDapp",
});

const valid = await verifyMessage({
  address: account.address,
  message: "Login to MyDapp",
  signature,
});
```

**Use cases:**
- Wallet-based authentication (Sign-In with Ethereum)
- Off-chain voting
- Proving ownership of an address
- Gasless approvals (EIP-2612 permit)

### Signing Typed Data (EIP-712)

EIP-712 enables structured, human-readable signatures (like "Approve 100 USDC to Uniswap").

```javascript
// ethers.js v6
const domain = {
  name: "MyDapp",
  version: "1",
  chainId: 1,
  verifyingContract: "0xContractAddress...",
};

const types = {
  Transfer: [
    { name: "to", type: "address" },
    { name: "amount", type: "uint256" },
  ],
};

const value = {
  to: "0xRecipient...",
  amount: "1000000000000000000", // 1 ETH in wei
};

const signature = await wallet.signTypedData(domain, types, value);
```

**Use cases:**
- ERC-20 Permit (approve without gas)
- Meta-transactions
- Order signing (Uniswap X, 0x protocol)
- DAO voting (Snapshot)

## Safe (Gnosis Safe) Multisig

Safe is the most widely used multisig wallet on Ethereum. **Over $100B+ in assets secured** (as of 2026).

### What It Is

A Safe is a smart contract wallet requiring M-of-N signatures to execute transactions.

**Example:** 2-of-3 Safe
- 3 owners: Alice, Bob, Carol
- Any 2 must approve transactions
- If Alice loses her key, Bob + Carol can still control the funds

**Key features:**
- Multi-signature (M-of-N threshold)
- Transaction batching (execute multiple calls in one transaction)
- Module system (plugins for advanced features)
- Time-locked transactions
- Spending limits per owner
- Web UI at https://app.safe.global

### Key Contract Addresses (v1.4.1, latest stable)

| Network | Safe Singleton | Safe Proxy Factory | MultiSend |
|---------|---------------|-------------------|-----------|
| Mainnet | `0x41675C099F32341bf84BFc5382aF534df5C7461a` | `0x4e1DCf7AD4e460CfD30791CCC4F9c8a4f820ec67` | `0x38869bf66a61cF6bDB996A6aE40D5853Fd43B526` |
| Arbitrum | `0x41675C099F32341bf84BFc5382aF534df5C7461a` | `0x4e1DCf7AD4e460CfD30791CCC4F9c8a4f820ec67` | `0x38869bf66a61cF6bDB996A6aE40D5853Fd43B526` |
| Base | `0x41675C099F32341bf84BFc5382aF534df5C7461a` | `0x4e1DCf7AD4e460CfD30791CCC4F9c8a4f820ec67` | `0x38869bf66a61cF6bDB996A6aE40D5853Fd43B526` |

(Safe uses deterministic deployment — same addresses across networks)

### Creating a Safe Programmatically

```javascript
// Using Safe SDK (@safe-global/protocol-kit)
import Safe from "@safe-global/protocol-kit";
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider("https://eth.llamarpc.com");
const owner1 = new ethers.Wallet("0xOwner1PrivateKey...", provider);

// Deploy a new Safe
const safe = await Safe.init({
  provider: "https://eth.llamarpc.com",
  signer: owner1.privateKey,
  safeAccountConfig: {
    owners: [
      "0xOwner1Address...",
      "0xOwner2Address...",
      "0xOwner3Address...",
    ],
    threshold: 2, // 2 of 3
  },
});

const safeAddress = await safe.getAddress();
console.log("Safe deployed at:", safeAddress);
```

### Using an Existing Safe

```javascript
import Safe from "@safe-global/protocol-kit";

// Connect to existing Safe
const safe = await Safe.init({
  provider: "https://eth.llamarpc.com",
  signer: owner1PrivateKey,
  safeAddress: "0xYourSafeAddress...",
});

// Create a transaction
const safeTransaction = await safe.createTransaction({
  transactions: [
    {
      to: "0xRecipient...",
      value: ethers.parseEther("1.0").toString(),
      data: "0x",
    },
  ],
});

// Sign with owner 1
const signedTx = await safe.signTransaction(safeTransaction);

// If threshold is met (e.g., already have 2 signatures), execute
const txResponse = await safe.executeTransaction(signedTx);
console.log("Transaction hash:", txResponse.hash);
```

### Batched Transactions (MultiSend)

One of Safe's most powerful features: execute multiple transactions atomically.

```javascript
// Batch multiple transactions
const safeTransaction = await safe.createTransaction({
  transactions: [
    {
      to: "0xUSDC...",
      value: "0",
      data: approveCalldata, // ERC-20 approve
    },
    {
      to: "0xUniswap...",
      value: "0",
      data: swapCalldata, // Uniswap swap
    },
    {
      to: "0xRecipient...",
      value: ethers.parseEther("0.1").toString(),
      data: "0x", // ETH transfer
    },
  ],
});

// All 3 execute atomically (all succeed or all revert)
```

**Use cases:**
- Approve + swap in one transaction
- Deploy contracts + initialize them
- Claim rewards + reinvest
- Complex DeFi strategies

### Safe for AI Agents

**Pattern:** Use a Safe with the agent as one owner + human as backup owner(s).

Example: 1-of-2 Safe
- Owner 1: Agent's wallet (hot wallet, automated)
- Owner 2: Human's wallet (cold wallet, recovery)
- Threshold: 1 (agent can act alone)

**Benefits:**
- If agent's key is compromised, human can remove it
- Human can always recover funds
- Agent can batch transactions for efficiency

## Connecting to dApps

### For an Agent Using a Browser

If the agent is automating a browser (e.g., with OpenClaw's browser tool):

1. **Wallet injects `window.ethereum`** (MetaMask, Rainbow, Coinbase Wallet)
2. **dApp calls `eth_requestAccounts`** to connect
3. **Wallet prompts user (or agent) to approve** the connection
4. **Once connected, dApp can request signatures and transactions**

```javascript
// In browser console or automation script
const accounts = await window.ethereum.request({
  method: "eth_requestAccounts",
});
console.log("Connected:", accounts[0]);

// Send transaction via injected wallet
const txHash = await window.ethereum.request({
  method: "eth_sendTransaction",
  params: [{
    from: accounts[0],
    to: "0xRecipient...",
    value: "0x0", // 0 ETH
    data: "0x...", // contract call data
  }],
});
```

**Guardrail:** The wallet UI shows the user what they're signing. This is a security feature. Don't bypass it unless the human explicitly opts into headless mode.

### For an Agent Using Scripts (Headless)

If the agent is running scripts (Node.js, Python, etc.) without a browser:

- Use ethers.js, viem, web3.js, or web3.py with a private key directly
- No browser needed — interact with contracts via RPC
- This is more efficient for agents but requires careful key management

```javascript
// ethers.js v6 - direct contract interaction
const provider = new ethers.JsonRpcProvider("https://eth.llamarpc.com");
const wallet = new ethers.Wallet("0xPrivateKey...", provider);

const contract = new ethers.Contract(contractAddress, abi, wallet);
const tx = await contract.transfer("0xRecipient...", ethers.parseEther("1.0"));
await tx.wait();
```

## Private Key Management for AI Agents

**This is the most critical security consideration.**

### Storage Options (Worst to Best)

❌ **NEVER:** Plain text in code or logs
```javascript
const privateKey = "0x1234..."; // ❌ DON'T DO THIS
```

❌ **NEVER:** Environment variables in shared environments
```bash
export PRIVATE_KEY=0x1234... # ❌ NOT SAFE
```

❌ **NEVER:** Committed to Git
```javascript
// config.js
module.exports = { privateKey: "0x1234..." }; // ❌ NEVER COMMIT
```

⚠️ **Acceptable for testing only:** Environment variables on local machine
```bash
# .env (local, never committed)
PRIVATE_KEY=0x1234...
```

✅ **Better:** Encrypted keystore (requires password to decrypt)
```javascript
// ethers.js v6
import { Wallet } from "ethers";
import fs from "fs";

// Encrypt wallet with password
const wallet = new Wallet("0xPrivateKey...");
const keystoreJson = await wallet.encrypt("SecurePassword123");
fs.writeFileSync("keystore.json", keystoreJson);

// Decrypt later
const decrypted = await Wallet.fromEncryptedJson(
  fs.readFileSync("keystore.json", "utf8"),
  "SecurePassword123"
);
```

✅ **Best for agents:** Hardware wallet or secure enclave (if available)
- Ledger, Trezor (agent can request signatures, keys never leave device)
- Cloud KMS (AWS KMS, GCP KMS, Azure Key Vault)
- Trusted execution environments (TEE)

✅ **Best for users:** Let them keep their keys in MetaMask/Rainbow/etc.
- Agent interacts through wallet's UI
- Keys never leave the user's control

### Key Generation Best Practices

1. **Use a cryptographically secure random number generator**
   ```javascript
   // ✅ Good (ethers.js uses crypto.getRandomValues)
   const wallet = Wallet.createRandom();
   
   // ❌ Bad (Math.random is NOT cryptographically secure)
   const badKey = "0x" + Math.random().toString(16).slice(2);
   ```

2. **Securely store the mnemonic (12 or 24 words)**
   - Mnemonics are human-readable backups of private keys
   - If the agent's database is lost, the mnemonic recovers the key
   - Store encrypted, never in plain text

3. **Use hierarchical deterministic (HD) wallets for multiple keys**
   ```javascript
   // Derive multiple wallets from one mnemonic (BIP-44)
   const wallet0 = Wallet.fromPhrase(mnemonic, "m/44'/60'/0'/0/0");
   const wallet1 = Wallet.fromPhrase(mnemonic, "m/44'/60'/0'/0/1");
   const wallet2 = Wallet.fromPhrase(mnemonic, "m/44'/60'/0'/0/2");
   ```

## Guardrails for AI Agents

### CRITICAL RULES

1. **NEVER extract a private key from MetaMask or any wallet without explicit human permission.**
   - The human must specifically say "yes, you can export my private key"
   - Even then, confirm they understand the risks

2. **NEVER store private keys in:**
   - Chat history or conversation logs
   - Plain text files
   - Environment variables in shared environments
   - Git repositories or version control
   - Databases without encryption
   - Any location accessible to other users

3. **NEVER move funds without human confirmation.** Always show:
   - The amount being sent
   - The destination address (verify it's checksummed)
   - The estimated gas cost
   - What the transaction does
   - Wait for explicit "yes, send it" from the human

4. **Prefer the wallet's native UI for signing** unless the human explicitly opts into CLI/scripting.
   - The wallet UI shows the human what they're signing
   - This prevents phishing and malicious signatures

5. **For agent-controlled wallets:** Use a dedicated wallet with limited funds.
   - Never use the human's main wallet for automated operations
   - If the agent's key is compromised, losses are limited

6. **Double-check addresses.**
   - Ethereum addresses are case-insensitive but should be checksummed
   - A single wrong character sends funds to the wrong place **permanently**
   - Use ENS names when possible and verify the resolved address
   ```javascript
   // Verify checksum
   const checksummed = ethers.getAddress("0xabcdef..."); // throws if invalid
   ```

7. **Test on a testnet first.**
   - Before any mainnet transaction, test the same flow on Sepolia
   - Or use a local Hardhat/Anvil fork
   - Testnet ETH is free (from faucets)

8. **Implement spending limits**
   - Set daily/weekly limits on agent-controlled wallets
   - Require human approval for transactions above threshold
   - Use a Safe multisig for high-value operations

9. **Log all transactions (but not keys)**
   - Keep an audit trail of what the agent sent and when
   - Never log private keys or mnemonics

10. **Assume keys will be compromised**
    - Design systems where a compromised agent key doesn't mean total loss
    - Use multisigs, recovery mechanisms, spending limits

### Example: Safe Transaction Flow

```javascript
async function sendSafely(wallet, to, value) {
  // 1. Validate destination address
  const checksummedTo = ethers.getAddress(to); // throws if invalid
  
  // 2. Check if destination is a known address
  const isKnown = await checkAddressDatabase(checksummedTo);
  if (!isKnown) {
    console.warn(`⚠️ Sending to unknown address: ${checksummedTo}`);
    // Optionally require human confirmation
  }
  
  // 3. Estimate gas and cost
  const tx = { to: checksummedTo, value };
  const gasEstimate = await wallet.estimateGas(tx);
  const feeData = await wallet.provider.getFeeData();
  const gasCost = gasEstimate * feeData.maxFeePerGas;
  const totalCost = value + gasCost;
  const totalCostUSD = Number(ethers.formatEther(totalCost)) * ETH_PRICE_USD;
  
  // 4. Get human approval if significant
  if (totalCostUSD > 10) {
    console.log(`This will cost $${totalCostUSD.toFixed(2)} total.`);
    console.log(`Sending ${ethers.formatEther(value)} ETH to ${checksummedTo}`);
    console.log(`Gas cost: ${ethers.formatEther(gasCost)} ETH`);
    
    const approved = await getHumanApproval();
    if (!approved) {
      console.log("Transaction cancelled by human.");
      return null;
    }
  }
  
  // 5. Send transaction
  const txResponse = await wallet.sendTransaction(tx);
  console.log(`✅ Transaction sent: ${txResponse.hash}`);
  
  // 6. Wait for confirmation
  const receipt = await txResponse.wait();
  console.log(`✅ Confirmed in block ${receipt.blockNumber}`);
  
  // 7. Log to audit trail (but not the private key!)
  logTransaction({
    hash: txResponse.hash,
    from: wallet.address,
    to: checksummedTo,
    value: ethers.formatEther(value),
    gasUsed: receipt.gasUsed.toString(),
    blockNumber: receipt.blockNumber,
    timestamp: Date.now(),
  });
  
  return receipt;
}
```

## Advanced Topics

### Session Keys (Temporary Permissions)

Session keys are temporary private keys with limited permissions. Example:
- "Can spend up to 1 ETH on Uniswap for the next 24 hours"
- "Can transfer NFTs from this collection"

**Implementation:**
1. Main wallet signs a permission for the session key
2. Session key signs transactions
3. Contract validates: is session key authorized + within limits?

**Use cases:**
- Gaming (agent can make in-game transactions without asking)
- Trading bots (limited trading permissions)
- Subscription payments (recurring payments without approval)

**Libraries:**
- Kernel (account abstraction)
- Web3Auth
- ZeroDev

### Social Recovery

Smart contract wallets can implement recovery mechanisms:

**Guardian-based recovery:**
- Set 3-5 trusted guardians (friends, family, other devices)
- If main key is lost, M-of-N guardians can approve a new owner

**Time-locked recovery:**
- Initiate recovery (e.g., "replace owner with new key")
- Wait 48 hours (time lock)
- If original owner doesn't cancel, recovery completes

**Examples:**
- Argent wallet (guardian-based)
- Safe (owner replacement with existing owners)

### EIP-2612: Permit (Gasless ERC-20 Approvals)

EIP-2612 enables approving ERC-20 transfers without gas:

```javascript
// Instead of calling approve() (costs gas)
// Sign a permit (free, off-chain)
const permit = {
  owner: wallet.address,
  spender: "0xSpenderAddress...",
  value: ethers.parseUnits("100", 6), // 100 USDC
  nonce: await token.nonces(wallet.address),
  deadline: Math.floor(Date.now() / 1000) + 3600, // 1 hour
};

const signature = await wallet.signTypedData(domain, types, permit);

// Spender calls permit() with signature (they pay gas)
await token.permit(
  permit.owner,
  permit.spender,
  permit.value,
  permit.deadline,
  signature.v,
  signature.r,
  signature.s
);
```

**Benefits:**
- User doesn't need ETH for gas
- App can sponsor the approval transaction
- Better UX (one transaction instead of approve + transfer)

**Supported tokens:**
- USDC, USDT (on some chains), DAI, most new ERC-20s

## Further Reading

- **Ethereum.org:** https://ethereum.org/en/wallets/
- **Safe documentation:** https://docs.safe.global/
- **ERC-4337 specification:** https://eips.ethereum.org/EIPS/eip-4337
- **EIP-7702 specification:** https://eips.ethereum.org/EIPS/eip-7702
- **Wallet security best practices:** https://ethereum.org/en/security/
