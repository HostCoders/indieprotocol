# 2. Protocol Architecture

## 2.1 Overview

The Indie Protocol is a set of 12 production smart contracts that together provide a complete economic layer for digital services. The contracts are organized into four functional groups:

```
┌─────────────────────────────────────────────────────────────────┐
│                    INDIE PROTOCOL (On-Chain)                     │
│                                                                  │
│  ┌──── SERVICE LAYER ────┐  ┌──── MARKETPLACE LAYER ────┐      │
│  │ ServiceNFT   (ERC-721)│  │ Marketplace               │      │
│  │ ServiceFactory        │  │ ServicePass    (ERC-1155)  │      │
│  │ ServiceRegistry       │  │ ServiceAccountImpl(ERC-6551│)     │
│  └───────────────────────┘  └────────────────────────────┘      │
│                                                                  │
│  ┌──── PAYMENT LAYER ────┐  ┌──── INFRASTRUCTURE ────────┐     │
│  │ InUSDToken    (ERC-20)│  │ PlatformConfig             │      │
│  │ InUSDCTreasury        │  │ BillingExecutor            │      │
│  │ TokenExchange         │  │ X402Receiver               │      │
│  └───────────────────────┘  └────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
                          │
                    Executor-Agnostic Interface
                          │
┌─────────────┬───────────┴───────────┬─────────────────┐
│ AI Agents   │ Workflow Engines      │ API Gateways    │
│ Compute     │ Data Pipelines        │ Any Service...  │
└─────────────┴───────────────────────┴─────────────────┘
```

## 2.2 Service Layer

### 2.2.1 ServiceNFT (ERC-721 + ERC-5643)

Each service deployed through the protocol is represented by a **dedicated ServiceNFT contract**. The contract itself is the service; individual token IDs are instances (licenses) of that service.

```
ServiceNFT Contract = "Image Generation API" (the service)
├── Token #1 = Instance owned by Alice
├── Token #2 = Instance owned by Bob
└── Token #3 = Instance owned by Charlie
```

**Key properties (immutable at deployment):**

| Property | Type | Description |
|----------|------|-------------|
| `creator` | address | Service creator, receives execution payments |
| `purchasePrice` | uint256 | One-time price to acquire an instance (in inUSD) |
| `executionPrice` | uint256 | Per-execution cost (in inUSD) |
| `maxExecutionPrice` | uint256 | Circuit breaker ceiling for execution cost |
| `subscriptionPrice` | uint256 | 30-day subscription price for unlimited executions |

**Subscription support (ERC-5643):**

ServiceNFTs natively support the ERC-5643 Subscription NFT standard. Token holders can:
- `renewSubscription(tokenId, duration)` --- Pay `subscriptionPrice` in inUSD directly to the creator, extending access by 30 days
- `cancelSubscription(tokenId)` --- Set expiry to zero
- `expiresAt(tokenId)` --- Query current subscription expiry

Subscriptions auto-cancel on transfer, ensuring new owners start fresh.

**Instance locking:**

Instances support a lock/unlock mechanism for execution safety:
- Owner can `lock(tokenId)` to signal the executor has control
- Executor can `unlock(tokenId)` when execution completes
- Locked tokens cannot be transferred (prevents mid-execution ownership changes)

**Agent identity (ERC-8004):**

Each ServiceNFT can optionally link to an `agentId` via `setAgentId()`, enabling machine-readable agent discovery per the ERC-8004 standard.

### 2.2.2 ServiceFactory

The ServiceFactory deploys new ServiceNFT contracts using **gasless EIP-712 typed signatures**:

```
1. Creator signs EIP-712 message: { serviceHash, prices, deadline, nonce }
2. Relayer submits ServiceFactory.registerWithAuth(creator, sig, prices)
3. Factory verifies signature, deploys new ServiceNFT contract
4. New service registered in ServiceRegistry
```

This means service creators never need ETH to register a service. The relayer pays gas; the creator only signs a message.

Sequential nonces per creator prevent replay attacks. The `serviceHash` links the on-chain registration to off-chain metadata (stored in any backend --- KV, IPFS, database).

### 2.2.3 ServiceRegistry

The ServiceRegistry is the canonical index of all services deployed through the protocol:

- Tracks which factory deployed each service (only authorized factories can register)
- Maps creators to their services (paginated queries)
- Maps users to owned instances (updated automatically by ServiceNFT on transfer/mint)
- Validates service authenticity for the Marketplace

**Factory authorization**: Only whitelisted factory contracts can register new services, preventing unauthorized contract registration.

## 2.3 Marketplace Layer

### 2.3.1 Marketplace

The Marketplace contract handles service purchasing, listing, and promotion:

**Purchasing** --- Two modes:
- `purchaseWithAuth()` --- Gasless: Relayer submits buyer's EIP-3009 signed token authorization. inUSD transfers directly from buyer to creator. ServiceNFT mints instance to buyer.
- `purchaseInstance()` --- Direct: Buyer pre-approves inUSD, calls purchase directly.

In both cases, **100% of the purchase price goes to the creator**. The protocol takes zero commission on service sales.

**Listing** --- Requires burning a Listing Pass (ServicePass ID 0):
```
Creator acquires Listing Pass → Relayer calls Marketplace.listService()
    → ServicePass.burn(creator, LISTING_PASS_ID)
    → ServiceNFT.setPaused(false)
    → Service is now discoverable
```

**Boosting** --- Burns Feature Passes (ServicePass ID 1) for promotional placement:
```
Marketplace.boostService(service, durationDays)
    → Burns 1 Feature Pass per day (1-30 days max)
    → Service appears in "Featured" section
```

**CQRS Analytics** --- The Marketplace tracks per-creator and per-buyer aggregate statistics on-chain for dashboards:
- `creatorSalesRevenue` / `creatorSalesCount`
- `buyerPurchaseSpent` / `buyerPurchaseCount`

### 2.3.2 ServicePass (ERC-1155)

ServicePass is a multi-token contract that gates marketplace actions:

| Pass ID | Name | Price | Purpose |
|---------|------|-------|---------|
| 0 | Listing Pass | Set by admin | Required to list a service on the marketplace |
| 1 | Feature Pass | Set by admin | Required to boost a service (1 per day) |

Passes are purchased with inUSD (revenue to `feeRecipient`) and burned on use. New pass types can be created by the admin for future marketplace features.

### 2.3.3 ServiceAccountImpl (ERC-6551 Token-Bound Account)

Every ServiceNFT instance automatically gets a deterministic smart contract wallet via ERC-6551. This is the **"fund once, execute many"** primitive:

```
TBA Address = f(chainId, serviceNFT_address, tokenId, implementation, salt)
```

**Dual authorization model:**

| Instance State | Authorized Actor | Permitted Actions |
|----------------|------------------|-------------------|
| **Locked** | Executor | Transfer inUSD to: Creator, FeeRecipient, or Treasury only |
| **Unlocked** | NFT Owner | Any transaction (withdraw, DeFi, etc.) |

When a service is being executed, the instance is locked and only the platform executor can move funds --- restricted to three whitelisted recipients. When idle, the owner has full control over the TBA, including withdrawing funds or interacting with any contract.

**Batch execution**: `batchExecute(calls[])` enables multiple transfers in a single transaction for gas efficiency during settlement.

**ERC-1271 signatures**: The TBA supports smart contract signatures (only when unlocked), enabling integration with protocols that verify contract-based signers.

## 2.4 Payment Layer

### 2.4.1 InUSDToken (ERC-20)

inUSD is the protocol's unit of account --- a stablecoin pegged 1:1 to USDC with 6 decimals.

**Minting controls:**
- Only addresses with `MINTER_ROLE` can mint (granted to the Treasury)
- Per-transaction cap: $1,000,000 (flash loan protection)
- Daily cap: $10,000,000 (systemic risk protection)
- 90% USDC backing required at mint time
- Pausable by admin in emergencies

**EIP-3009 support**: inUSD implements `transferWithAuthorization`, enabling gasless transfers where the token holder signs an off-chain authorization and any relayer can submit it.

### 2.4.2 InUSDCTreasury

The Treasury is the USDC reserve vault that backs inUSD:

**Deposit (USDC → inUSD):**
```
User deposits 100 USDC
    → 90 USDC stays as reserves
    → 10 USDC sent to revenueBeneficiary (protocol revenue)
    → 100 inUSD minted to user (1:1)
```

**Redemption (inUSD → USDC):**
```
User redeems 100 inUSD
    → 100 inUSD burned
    → 90 USDC sent to user (90% redemption rate)
```

The 10% spread is the protocol's primary revenue mechanism. It is collected at deposit time, meaning the treasury always holds exactly 90% of outstanding inUSD supply in USDC reserves.

**Minimum withdrawal**: $1 (prevents dust attacks).

### 2.4.3 TokenExchange

TokenExchange provides the gasless swap interface:

- `swapUSDCWithAuth()` --- USDC→inUSD via EIP-3009 (1:1, zero fee)
- `swapInUSDWithAuth()` --- inUSD→USDC via EIP-3009 (90% out, 10% fee)
- Amount bounds: $1 minimum, $500,000 maximum per swap

The exchange requires `RELAYER_ROLE` to submit transactions, ensuring all swaps flow through authorized relayers.

## 2.5 Infrastructure Layer

### 2.5.1 PlatformConfig

PlatformConfig is the **single source of truth** for all protocol contract addresses. Every contract reads its dependencies from PlatformConfig rather than hardcoding addresses.

**Stored addresses:**

| Key | Contract |
|-----|----------|
| EXECUTOR | Off-chain executor address |
| PRODUCT_REGISTRY | ServiceRegistry |
| MARKETPLACE | Marketplace |
| BILLING_EXECUTOR | BillingExecutor |
| PRODUCT_FACTORY | ServiceFactory |
| USDC | USDC token |
| TOKEN_EXCHANGE | TokenExchange |
| FEE_RECIPIENT | Protocol fee collection |
| INUSD_TREASURY | InUSDCTreasury |
| SERVICE_ACCOUNT_IMPL | TBA implementation |
| X402_RECEIVER | X402Receiver |
| SERVICE_PASS | ServicePass |

This design enables atomic upgrades: change an address in PlatformConfig, and all dependent contracts immediately use the new address.

### 2.5.2 BillingExecutor

BillingExecutor records aggregated execution billing statistics on-chain. It serves as the audit trail for off-chain execution settlements:

```
Executor calls recordStats(creators[], amounts[], executions[], services[])
    → Batch ID assigned
    → Per-creator totals updated (totalPaid, totalExecutions)
    → Per-service totals updated (totalEarnings, totalExecutions)
    → Event: Settled(batchId, totalAmount, executionCount)
```

Maximum batch size: 250 entries. Only the authorized executor can record stats.

This contract does **not** perform settlement --- actual fund transfers happen through TBAs. BillingExecutor provides the on-chain proof that settlement occurred, enabling dashboards, dispute resolution, and auditing.

### 2.5.3 X402Receiver

X402Receiver supports the HTTP 402 payment protocol. It receives USDC payments from x402 facilitator settlements and holds them until the admin withdraws to the treasury for inUSD conversion.

This enables a powerful integration pattern: services can monetize via standard HTTP 402 "Payment Required" responses, with settlement flowing through the Indie Protocol's payment infrastructure.

## 2.6 Contract Relationship Map

```
                        PlatformConfig
                     (Central Registry)
                            │
          All contracts read dependencies from here
                            │
    ┌───────────┬───────────┼───────────┬───────────────┐
    │           │           │           │               │
    ▼           ▼           ▼           ▼               ▼
ServiceFactory  Marketplace BillingExec ServicePass  TokenExchange
    │              │           │           │              │
    │ deploys      │ purchases │ records   │ burns        │ swaps
    ▼              ▼           │           │              ▼
ServiceNFT ◄──────┘           │           │        InUSDCTreasury
    │                         │           │              │
    │ tracks ownership        │           │              │ mints/burns
    ▼                         │           │              ▼
ServiceRegistry               │           │         InUSDToken
    ▲                         │           │              │
    │ addOwnership            │           │              │
    │ removeOwnership         │           │              │
    │                         │           │              │
ServiceAccountImpl ◄──────────┘───────────┘──────────────┘
(TBA: transfers inUSD to creator/fee/treasury)
```

## 2.7 Multi-Chain Design

The protocol supports multi-chain deployment through:

- **PlatformConfig per chain**: Each chain deployment has its own PlatformConfig with chain-specific addresses
- **Chain ID isolation**: Off-chain storage keys use `${chainId}:` prefixes to prevent cross-chain data collision
- **X-Chain-ID header**: Executor backends route requests to the correct chain based on HTTP headers

Currently deployed on Base Sepolia (testnet), with Base Mainnet as the production target and Arbitrum Sepolia as a secondary testnet.
