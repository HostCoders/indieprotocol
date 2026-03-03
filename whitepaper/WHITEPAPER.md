# The Indie Protocol

## A General-Purpose Smart Contract Framework for Service Tokenization, Marketplace, and Billing

**Whitepaper v1.1 | March 2026**

---

> *Own your service. Set your price. Get paid per execution.*

---

**Abstract**

The Indie Protocol is a modular, EVM-based smart contract framework that enables any digital service to be tokenized as an NFT, listed on a decentralized marketplace, and billed through transparent on-chain mechanisms --- including per-execution micropayments, one-time purchases, and recurring subscriptions.

The protocol is **executor-agnostic**. It defines the ownership, discovery, billing, and settlement layers for digital services without prescribing how those services run. Any execution backend --- AI agents, API gateways, SaaS platforms, workflow engines, compute providers --- can plug into the Indie Protocol to gain NFT-based ownership, gasless marketplace operations, and trustless payment rails.

At its core, the protocol introduces:

- **Service-as-NFT** (ERC-721 + ERC-5643): Every service is a contract. Every instance is a token. Subscriptions are native.
- **Agent Wallets** (ERC-6551): Each service instance gets a dedicated Token-Bound Account for "fund once, execute many" billing.
- **inUSD Stablecoin** (ERC-20): A USDC-backed platform currency with free entry and 10% exit fee, incentivizing ecosystem retention.
- **Gasless Marketplace**: Fully gasless listing, purchasing, and billing via EIP-712, EIP-3009, and ERC-4337.
- **Composable Billing**: Per-execution, subscription (30-day), one-time purchase, and pass-based access models in a single framework.

The Indie Protocol is designed for builders who want to own and monetize their services, platforms that want plug-and-play service economics, and ecosystems that need trustless service-to-service billing.

---

| | |
|---|---|
| **Protocol** | Indie Protocol |
| **Currency** | inUSD (ERC-20, USDC-backed, 6 decimals) |
| **Chain** | Base (Ethereum L2), multi-chain capable |
| **Standards** | ERC-721, ERC-1155, ERC-5643, ERC-6551, ERC-8004, EIP-712, EIP-3009, EIP-4337, x402 |
| **Contracts** | 12 production contracts, Solidity 0.8.20, OpenZeppelin v5 |
| **Status** | Testnet (Base Sepolia), 1,000+ automated tests |

---

*This document describes the architecture, economics, and design rationale of the Indie Protocol. It is intended for developers building on the protocol, investors, grant evaluators, and ecosystem participants seeking to understand the service tokenization layer.*
# 1. Introduction

## 1.1 The Service Economy Problem

Digital services --- APIs, AI agents, automation workflows, compute tasks, data pipelines, consulting deliverables --- form the backbone of the modern economy. Yet the infrastructure for owning, trading, and billing these services remains fragmented and centralized:

- **Ownership is platform-dependent.** A developer's API, agent, or workflow exists at the pleasure of the platform hosting it. There is no portable, verifiable ownership primitive for digital services.
- **Billing is rigid.** Most platforms force a single model: monthly subscriptions, per-seat licensing, or flat-rate APIs. There is no unified framework that supports per-execution micropayments, subscriptions, and one-time purchases simultaneously.
- **Discovery is siloed.** Services live in walled-garden marketplaces (AWS Marketplace, Zapier, RapidAPI) with platform-specific listing, purchasing, and settlement flows.
- **Composability is absent.** Service A cannot trustlessly call and pay Service B without both being on the same platform with the same billing system.

The Indie Protocol addresses these structural gaps with a general-purpose smart contract framework that any service --- regardless of its execution environment --- can use for tokenization, marketplace access, and on-chain billing.

## 1.2 What the Indie Protocol Is

The Indie Protocol is a **set of composable smart contracts** deployed on EVM-compatible chains that provide:

1. **Service Tokenization**: Mint any digital service as an ERC-721 NFT with built-in subscription support (ERC-5643). Each instance of the service is a token. Ownership is transferable, verifiable, and composable.

2. **Decentralized Marketplace**: List, discover, purchase, and boost services through an on-chain marketplace with gasless UX. No intermediary controls listing or delisting.

3. **Flexible Billing**: Three billing models in one framework --- per-execution (metered), subscription (30-day recurring), and one-time purchase --- all settled in inUSD.

4. **Token-Bound Accounts**: Each service instance gets a dedicated ERC-6551 wallet. Users fund the wallet; the executor draws from it per execution. Full owner control when idle.

5. **Payment Infrastructure**: inUSD stablecoin (1:1 USDC-backed) with gasless swaps, providing a stable unit of account for all protocol operations.

6. **Access Gating**: ERC-1155 ServicePass tokens gate marketplace actions (listing, boosting), creating a spam-resistant, fee-based access model.

## 1.3 What the Indie Protocol Is Not

The protocol explicitly does **not** prescribe:

- **How services execute.** The protocol is executor-agnostic. An AI agent platform, a workflow engine, an API gateway, or a human services marketplace can all plug into the same contracts.
- **Where services run.** Services can run on Cloudflare Workers, AWS Lambda, bare metal, or a developer's laptop. The protocol handles ownership and billing; execution is external.
- **What services do.** From LLM inference to data transformation to image generation to legal document review --- any digital service that can report execution events can use the protocol.

This separation of concerns is fundamental to the protocol's design. The smart contracts define the **economic layer** for services; execution environments are pluggable implementations.

## 1.4 Design Principles

| Principle | Description |
|-----------|-------------|
| **Executor-Agnostic** | The protocol defines ownership, billing, and marketplace. Execution is pluggable. Any backend can integrate. |
| **Creator-First Economics** | Service creators receive 100% of execution and purchase revenue with zero commission. The protocol sustains itself through ecosystem-level fees (10% exit fee, $0.0001 execution micro-fee), not creator revenue deductions. |
| **Gasless by Default** | All user-facing operations (register, list, purchase, swap) are gasless via EIP-712, EIP-3009, and ERC-4337 paymasters. |
| **Composable Primitives** | Each contract is independently useful. Use ServiceNFT without the Marketplace. Use inUSD without ServiceNFTs. Mix and match. |
| **Minimal Extraction** | Protocol sustainability through a 10% exit fee on USDC withdrawals and flat per-execution fees --- no rent-seeking on creator revenue. |
| **Standards-First** | Built on established ERCs (721, 1155, 5643, 6551, 8004) and EIPs (712, 3009, 4337) for maximum interoperability. |

## 1.5 Example Integrations

To illustrate the protocol's generality, here are potential executor implementations:

| Executor Type | How It Uses the Protocol |
|---------------|--------------------------|
| **AI Agent Platform** | Each agent is a ServiceNFT. Buyers purchase instances, fund TBAs, and the platform executor bills per inference call. |
| **Workflow Automation Engine** | Each workflow template is a ServiceNFT. Buyers run workflows; per-execution billing deducts from the TBA. |
| **API Marketplace** | Each API endpoint is a ServiceNFT. Consumers purchase access; metered billing via TBA or subscription via ERC-5643. |
| **Compute Marketplace** | Each compute task type is a ServiceNFT. Jobs are billed per execution from the TBA. |
| **Consulting/Freelance Platform** | Each service offering is a ServiceNFT. One-time purchase model for deliverables. |
| **Data Pipeline Marketplace** | Each ETL pipeline is a ServiceNFT. Subscription model for continuous data feeds. |

## 1.6 Document Structure

This whitepaper is organized as follows:

| Section | Content |
|---------|---------|
| **2** | Protocol architecture and smart contract design |
| **3** | Token economics and fee structure (inUSD) |
| **4** | Service lifecycle: registration, listing, purchase, execution, settlement |
| **5** | Security model, access control, and trust assumptions |
| **6** | Standards and interoperability (ERCs, EIPs, x402) |
| **7** | Governance and decentralization roadmap |
| **8** | Competitive analysis and market positioning |
| **9** | Roadmap and milestones |
| **10** | Risk factors |
| **A** | Appendix: Smart contract reference |
| **B** | Appendix: Glossary |
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
# 3. Token Economics

## 3.1 inUSD: The Protocol Currency

inUSD is the Indie Protocol's native unit of account --- an ERC-20 stablecoin pegged 1:1 to USDC with 6 decimals. All protocol operations --- service purchases, execution billing, subscriptions, pass purchases --- are denominated and settled in inUSD.

### Why a Protocol-Specific Stablecoin?

| Reason | Explanation |
|--------|-------------|
| **Gasless transfers** | inUSD implements EIP-3009 `transferWithAuthorization`, enabling off-chain signed transfers. Native USDC on some chains may not support this. |
| **Ecosystem retention** | Free entry (USDC→inUSD at 1:1) and 10% exit fee incentivizes participants to keep funds in the ecosystem. |
| **Controlled minting** | Daily and per-transaction caps protect against flash loan attacks and systemic risk. |
| **Emergency controls** | Pausable transfers enable rapid response to security incidents. |
| **Accounting clarity** | All protocol analytics use a single denomination without external price dependencies. |

### Minting Mechanism

inUSD is minted exclusively through USDC deposits into the InUSDCTreasury:

```
Deposit 100 USDC
    → 90 USDC → Treasury reserves
    → 10 USDC → Revenue beneficiary (protocol revenue)
    → 100 inUSD minted to depositor
```

**Minting safeguards:**
- `MINTER_ROLE` required (only Treasury holds this role)
- Per-transaction maximum: $1,000,000
- Daily maximum: $10,000,000
- 90% USDC backing verified at each mint
- Pausable by admin

### Redemption Mechanism

inUSD is redeemed (burned) through the Treasury at a 90% rate:

```
Redeem 100 inUSD
    → 100 inUSD burned
    → 90 USDC returned to redeemer
```

The 10% spread was already collected at deposit time. The treasury always holds exactly 90% of outstanding inUSD supply in USDC, ensuring full solvency for all redemptions at the stated rate.

**Redemption safeguards:**
- `EXCHANGER_ROLE` required (only TokenExchange holds this role)
- Minimum withdrawal: $1 (dust prevention)
- Pausable by admin

### Supply Dynamics

```
inUSD Supply = Total USDC Deposited (cumulative)
               - Total inUSD Redeemed (cumulative)
               - Total inUSD Burned (fees, passes)

Treasury USDC = 90% × Total USDC Deposited
                - USDC paid out in redemptions
                - Admin withdrawals

Invariant: Treasury USDC ≥ 90% × inUSD Supply
```

## 3.2 Fee Structure

The Indie Protocol generates revenue through three mechanisms:

### 3.2.1 Exit Fee (Primary Revenue)

| Direction | Rate | Fee | Recipient |
|-----------|------|-----|-----------|
| USDC → inUSD (entry) | 1:1 | 0% | No fee |
| inUSD → USDC (exit) | 90:100 | 10% | Revenue beneficiary |

The asymmetric entry/exit creates a natural incentive to remain in the ecosystem. Participants who earn and spend inUSD within the protocol never pay this fee. Only when extracting value to external USDC does the 10% apply.

### 3.2.2 Execution Fee (Micro-Revenue)

Each service execution incurs a flat platform fee (configurable, default $0.0001) paid from the service instance's TBA to the protocol's fee recipient. This is independent of the creator's execution price.

At scale, execution fees compound:
- 10,000 daily executions = $1/day
- 1,000,000 daily executions = $100/day
- 100,000,000 daily executions = $10,000/day

### 3.2.3 ServicePass Sales (Access Revenue)

| Pass Type | Price | Purpose |
|-----------|-------|---------|
| Listing Pass (ID 0) | Set by admin | Required to list a service on the marketplace |
| Feature Pass (ID 1) | Set by admin | Required to boost a service for 1 day |

Pass revenue flows to the protocol's fee recipient. Future pass types can be created for additional marketplace features.

### 3.2.4 Revenue Flow Summary

```
Revenue Sources:
├── 10% Exit Fee on USDC withdrawals ──→ Revenue Beneficiary
├── Flat execution fee ($0.0001) ──────→ Fee Recipient
├── Listing Pass sales ────────────────→ Fee Recipient
└── Feature Pass sales ────────────────→ Fee Recipient

Creator Revenue (untouched by protocol):
├── 100% of service purchase price
├── 100% of execution fees (minus platform micro-fee)
└── 100% of subscription payments
```

## 3.3 Creator Economics

> **What "zero commission" means**: Creators receive 100% of service purchase payments, execution fees, and subscription revenue. The protocol charges no commission or percentage on these transactions. Separately, the protocol collects a flat $0.0001 per-execution micro-fee and participants who convert inUSD back to USDC pay a 10% exit fee. These are ecosystem-level fees, not deductions from creator revenue.

Creators receive revenue through three channels, **none of which are subject to protocol commission**:

### 3.3.1 Purchase Revenue

When a buyer purchases a service instance, the full `purchasePrice` (in inUSD) transfers directly from buyer to creator via EIP-3009. The Marketplace contract facilitates but does not intermediate the payment.

### 3.3.2 Execution Revenue

Per-execution billing draws from the buyer's TBA:
- Creator's `executionPrice` → Creator's address
- Platform fee ($0.0001) → Fee Recipient
- API costs (if applicable) → Configurable recipient

The creator sets `executionPrice` at service deployment (immutable). The `maxExecutionPrice` acts as a circuit breaker --- if projected execution cost exceeds this ceiling, execution halts immediately to protect the buyer.

### 3.3.3 Subscription Revenue

For services with `subscriptionPrice > 0`, buyers can subscribe for 30-day unlimited execution periods. The full `subscriptionPrice` transfers directly from buyer to creator via `renewSubscription()`.

### 3.3.4 Creator Revenue Example

```
Service: "AI Content Generator"
├── purchasePrice: 50 inUSD
├── executionPrice: 0.10 inUSD
├── subscriptionPrice: 25 inUSD/month

Scenario: 100 buyers, 50 subscribe, avg 200 executions/month each

Monthly Creator Revenue:
├── Purchases: 100 × 50 = 5,000 inUSD (one-time)
├── Subscriptions: 50 × 25 = 1,250 inUSD/month (recurring)
├── Executions: 50 × 200 × 0.10 = 1,000 inUSD/month (metered, non-subscribers)
└── Total monthly (after first month): 2,250 inUSD

Creator keeps: 100% of the above
Protocol takes: 50 × 200 × 0.0001 + 50 × 200 × 0.0001 = 2 inUSD/month in execution fees
```

## 3.4 Buyer Economics

Buyers interact with inUSD in three ways:

1. **Fund entry**: Deposit USDC to receive inUSD at 1:1 (free)
2. **Spend inUSD**: Purchase services, fund TBAs, subscribe --- all at face value
3. **Exit** (optional): Convert remaining inUSD back to USDC at 90% (10% fee)

The economic incentive is clear: buyers who spend inUSD on services pay zero protocol fees. The 10% exit fee only applies when withdrawing unused funds.

## 3.5 Ecosystem Flywheel

```
Free Entry (0% fee)
    → More USDC deposits
        → Larger inUSD supply
            → More services purchasable
                → More creators attracted
                    → Better services available
                        → More buyers attracted
                            → Cycle reinforces

Exit Fee (10%) discourages withdrawal
    → inUSD circulates within ecosystem
        → Higher protocol TVL
            → Greater ecosystem stability
```

## 3.6 Comparison to Alternatives

| Model | Creator Share | Buyer Fee | Settlement |
|-------|-------------|-----------|------------|
| Apple App Store | 70% | Included in price | Monthly, $100 minimum |
| Stripe | 97.1% | 2.9% + $0.30 | 2 business days |
| Ethereum gas | 100% (minus gas) | Gas fees ($0.50-$50) | Per transaction |
| **Indie Protocol** | **100% (0% commission)** | **0% (inUSD) / 10% exit** | **5-minute batch** |

The Indie Protocol charges zero commission on service revenue --- the highest creator share of any comparable marketplace model. Buyer costs are minimal for participants who transact within the ecosystem; the 10% exit fee applies only when converting inUSD back to USDC.
# 4. Service Lifecycle

## 4.1 Overview

Every service on the Indie Protocol follows a defined lifecycle from creation to execution. This section describes each phase and the contracts involved.

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ Register │──▶│   List   │──▶│ Purchase │──▶│  Fund &  │──▶│ Execute  │
│          │   │          │   │          │   │  Lock    │   │ & Settle │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
ServiceFactory  Marketplace   Marketplace    TBA (ERC-6551)  Executor
ServiceRegistry ServicePass   ServiceNFT     InUSDToken      BillingExecutor
```

## 4.2 Phase 1: Service Registration

**Actor**: Creator (Builder)
**Contracts**: ServiceFactory, ServiceRegistry
**Gas cost to creator**: Zero (gasless via EIP-712)

### Flow

1. Creator prepares service metadata off-chain (name, description, workflow/API specification)
2. Creator defines pricing:
   - `purchasePrice` --- one-time acquisition cost
   - `executionPrice` --- per-run cost
   - `maxExecutionPrice` --- circuit breaker ceiling
   - `subscriptionPrice` --- 30-day subscription cost (0 = disabled)
3. Creator signs an EIP-712 typed message containing `{ serviceHash, prices, deadline, nonce }`
4. Relayer submits to `ServiceFactory.registerWithAuth()`
5. Factory verifies signature, deploys a new ServiceNFT contract
6. Factory registers the new service in ServiceRegistry

### Outputs

- New ServiceNFT contract deployed (address deterministic from factory nonce)
- Service registered in ServiceRegistry with creator address and timestamp
- Service starts in **paused** state (not yet listed on marketplace)
- Event: `ServiceRegistered`, `ServiceDeployed`

### Security Properties

- Creator signature prevents unauthorized registration
- Sequential nonces prevent replay attacks
- Deadline prevents stale signature submission
- Only authorized factories can register in the ServiceRegistry

## 4.3 Phase 2: Service Listing

**Actor**: Creator
**Contracts**: Marketplace, ServicePass
**Prerequisite**: Creator holds a Listing Pass (ServicePass ID 0)

### Flow

1. Creator acquires a Listing Pass by purchasing from ServicePass contract (costs inUSD)
2. Creator requests listing via relayer
3. Relayer calls `Marketplace.listService(service)`
4. Marketplace burns one Listing Pass from the creator
5. ServiceNFT is unpaused (set to active)
6. Service is now discoverable on the marketplace

### Spam Prevention

The Listing Pass mechanism serves as a Sybil-resistant gating mechanism. Each listing costs a pass, which costs inUSD. This prevents spam listings while keeping the bar low enough for legitimate creators.

### Boosting (Optional)

Creators can promote their services into a "Featured" section:

1. Creator acquires Feature Passes (ServicePass ID 1)
2. Relayer calls `Marketplace.boostService(service, durationDays)`
3. Marketplace burns `durationDays` Feature Passes (1-30 days max)
4. Service receives promotional placement

### Delisting

Services can be delisted (paused) by the relayer via `Marketplace.delistService()`. Delisted services cannot be purchased but existing instances continue to function.

## 4.4 Phase 3: Service Purchase

**Actor**: Buyer (Producer)
**Contracts**: Marketplace, ServiceNFT, InUSDToken
**Gas cost to buyer**: Zero (gasless via EIP-3009)

### Gasless Purchase Flow

1. Buyer discovers service on the marketplace
2. Buyer signs EIP-3009 `transferWithAuthorization` for `purchasePrice` in inUSD
3. Relayer submits to `Marketplace.purchaseWithAuth(buyer, service, ...)`
4. Marketplace validates:
   - Service exists in ServiceRegistry
   - Service is not paused
   - Buyer has sufficient inUSD balance
5. inUSD transfers from buyer to creator (via EIP-3009 authorization)
6. `ServiceNFT.mintInstance(buyer)` mints new token to buyer
7. ServiceRegistry tracks the buyer's new ownership

### Direct Purchase Flow

For users who prefer on-chain interaction:

1. Buyer approves inUSD for Marketplace contract
2. Buyer calls `Marketplace.purchaseInstance(service)`
3. Same validation and minting flow as above

### Outputs

- New ERC-721 token minted to buyer (auto-incrementing token ID)
- TBA address deterministically derived for the new instance
- ServiceRegistry updated with buyer's ownership
- 100% of purchase price received by creator
- Event: `InstancePurchased`, `InstanceMinted`

## 4.5 Phase 4: Fund & Lock

**Actor**: Buyer
**Contracts**: ServiceAccountImpl (TBA), InUSDToken, ServiceNFT

### Funding the TBA

1. Buyer transfers inUSD to the TBA address (standard ERC-20 transfer)
2. TBA now holds funds for execution billing
3. No special function needed --- any inUSD transfer to the TBA address works

### Locking for Execution

1. Buyer calls `ServiceNFT.lock(tokenId)` to signal readiness for execution
2. While locked:
   - Only the Executor can move TBA funds
   - Executor is restricted to transferring inUSD to: Creator, FeeRecipient, or Treasury
   - Owner cannot transfer or interact via the TBA
   - Token cannot be transferred (prevents ownership changes during execution)
3. Executor can also call `ServiceNFT.executorLock(tokenId)` to initiate locking

### Unlocking

1. Executor calls `ServiceNFT.unlock(tokenId)` when execution completes or the instance goes idle
2. Owner regains full control of the TBA
3. Owner can withdraw remaining funds or refund the TBA

## 4.6 Phase 5: Execution & Settlement

**Actor**: Executor (external system)
**Contracts**: ServiceAccountImpl (TBA), BillingExecutor

### Execution Billing

The executor is any off-chain system authorized in PlatformConfig. When a service executes:

1. Executor calculates the execution cost based on `executionPrice` and any API costs
2. Executor verifies cost does not exceed `maxExecutionPrice` (circuit breaker)
3. Executor calls `TBA.execute()` or `TBA.batchExecute()` to transfer inUSD:

```
Execution billing breakdown:
├── executionPrice ──→ Creator address
├── Platform fee ────→ FeeRecipient address
└── API costs ───────→ Treasury or configured recipient
```

### Batch Settlement

For efficiency, executors can batch multiple billing events:

1. Executor accumulates execution records over a settlement window (e.g., 5 minutes)
2. Executor calls `BillingExecutor.recordStats()` with aggregated data:
   - creators[] --- list of creators with revenue in this batch
   - amounts[] --- inUSD amounts per creator
   - executions[] --- execution counts per creator
   - services[] --- service addresses per entry
3. BillingExecutor records the batch with a unique ID and updates per-creator and per-service aggregates
4. Event: `Settled(batchId, totalAmount, executionCount)`

### Settlement Guarantees

- BillingExecutor maximum batch size: 250 entries
- Only the authorized executor can record stats
- Batch IDs are sequential and immutable once recorded
- Per-creator and per-service totals are cumulative and auditable

## 4.7 Phase 6: Subscription Renewal

**Actor**: Buyer
**Contracts**: ServiceNFT, InUSDToken

For services with `subscriptionPrice > 0`:

1. Buyer calls `ServiceNFT.renewSubscription(tokenId, duration)` where duration = 30 days
2. `subscriptionPrice` in inUSD transfers from buyer to creator
3. Subscription expiry extends by 30 days from current time (or from current expiry if still active)
4. Event: `SubscriptionUpdate(tokenId, expiration)` (ERC-5643)

Executors can check `expiresAt(tokenId)` to verify active subscriptions before execution.

## 4.8 Lifecycle Summary

| Phase | Actor | Gas Cost | Key Contract | Output |
|-------|-------|----------|-------------|--------|
| Register | Creator | Zero | ServiceFactory | New ServiceNFT deployed |
| List | Creator | Zero | Marketplace + ServicePass | Service discoverable |
| Purchase | Buyer | Zero | Marketplace + ServiceNFT | Instance NFT minted |
| Fund | Buyer | Standard ERC-20 | TBA + InUSDToken | TBA funded with inUSD |
| Execute | Executor | Standard | TBA + BillingExecutor | Creator paid, stats recorded |
| Subscribe | Buyer | Standard | ServiceNFT | 30-day access renewed |
# 5. Security Model

## 5.1 Trust Model

The Indie Protocol operates with a clear separation between trustless on-chain guarantees and trusted off-chain components:

| Component | Trust Level | Rationale |
|-----------|-------------|-----------|
| Service ownership | **Trustless** | ERC-721 on-chain, immutable |
| Payment settlement | **Trustless** | inUSD transfers are on-chain ERC-20 operations |
| Pricing | **Trustless** | Immutable at deployment in ServiceNFT |
| TBA fund custody | **Trustless** | ERC-6551, owner controls when unlocked |
| Executor authorization | **Trusted** | Executor address set in PlatformConfig by admin |
| Service execution | **Trusted** | Off-chain; relies on executor to run services correctly |
| Metadata | **Trusted** | Stored off-chain; integrity depends on storage backend |
| Relayer | **Trusted** | Submits gasless transactions; cannot forge user signatures |

### The Executor Trust Assumption

The executor is the most significant trusted component. It is authorized to:
- Move TBA funds (only to whitelisted recipients, only when instance is locked)
- Record billing statistics in BillingExecutor
- Lock/unlock service instances

**Mitigations:**
- Executor can only transfer inUSD (no other tokens or ETH)
- Recipient whitelist: only Creator, FeeRecipient, or Treasury
- Transfers only permitted when instance is locked (owner opted in)
- All transfers are on-chain and auditable
- BillingExecutor creates an immutable audit trail

### The Relayer Trust Assumption

The relayer submits gasless transactions but **cannot forge actions**:
- All gasless operations require cryptographic signatures from the actual user
- EIP-712 signatures for service registration
- EIP-3009 signatures for token transfers
- The relayer can at most refuse to submit (liveness risk), never fabricate

## 5.2 Access Control

The protocol uses OpenZeppelin's AccessControl with role-based permissions across all contracts:

### Role Matrix

| Contract | Role | Granted To | Permissions |
|----------|------|------------|-------------|
| InUSDToken | `DEFAULT_ADMIN_ROLE` | Deployer/Multisig | Pause, configure, set caps |
| InUSDToken | `MINTER_ROLE` | InUSDCTreasury | Mint inUSD |
| InUSDToken | `TREASURY_ROLE` | InUSDCTreasury | Burn inUSD during redemption |
| InUSDCTreasury | `DEFAULT_ADMIN_ROLE` | Deployer/Multisig | Pause, withdraw, configure |
| InUSDCTreasury | `EXCHANGER_ROLE` | TokenExchange | Burn inUSD for USDC |
| TokenExchange | `RELAYER_ROLE` | Backend relayer | Execute gasless swaps |
| Marketplace | `RELAYER_ROLE` | Backend relayer | Execute gasless purchases, list/delist |
| ServiceFactory | `RELAYER_ROLE` | Backend relayer | Execute gasless registrations |
| ServicePass | `BURNER_ROLE` | Marketplace | Burn passes during listing/boosting |
| ServiceRegistry | `DEFAULT_ADMIN_ROLE` | Deployer/Multisig | Authorize/deauthorize factories |

### Custom Access Patterns

| Contract | Modifier | Logic |
|----------|----------|-------|
| ServiceNFT | `onlyMarketplace` | `msg.sender == platformConfig.marketplace()` |
| ServiceNFT | `onlyInstanceOwner` | `ownerOf(tokenId) == msg.sender` |
| ServiceNFT | `onlyExecutor` | `msg.sender == platformConfig.executor()` |
| BillingExecutor | `onlyExecutor` | `msg.sender == platformConfig.executor()` |
| ServiceRegistry | `onlyAuthorizedFactory` | `authorizedFactories[msg.sender]` |
| ServiceAccountImpl | Dual auth | Executor (locked) or Owner (unlocked) |

## 5.3 Security Guards

### Reentrancy Protection

All contracts handling external calls use OpenZeppelin's `ReentrancyGuard`:
- InUSDCTreasury
- TokenExchange
- Marketplace
- BillingExecutor
- ServiceFactory
- ServiceNFT (subscription renewal)
- X402Receiver
- ServiceAccountImpl (via state increment)

### Pausability

Emergency pause capability on critical contracts:
- **InUSDToken**: Pauses all transfers (freezes the economy)
- **InUSDCTreasury**: Pauses deposits and redemptions
- **TokenExchange**: Pauses USDC/inUSD swaps

Pause is restricted to `DEFAULT_ADMIN_ROLE` holders.

### Safe Token Transfers

All contracts use OpenZeppelin's `SafeERC20` for ERC-20 interactions, preventing:
- Silent transfer failures (reverts if transfer returns false)
- Non-standard token behavior

### Checks-Effects-Interactions (CEI)

All contracts follow the CEI pattern. For example, ServiceFactory increments nonces before deploying contracts, preventing reentrancy via constructor callbacks.

### Flash Loan Protection

InUSDToken enforces:
- Per-transaction mint cap: $1,000,000
- Daily mint cap: $10,000,000

These limits prevent economic attacks using flash-loaned USDC to mint large quantities of inUSD.

## 5.4 Attack Surface Analysis

### Service Purchase Reentrancy

**Vector**: Malicious ServiceNFT contract with reentrant `onERC721Received` callback.
**Mitigation**: `ReentrancyGuard` on Marketplace. Tested with `MaliciousReentrantBuyer` contract.

### Fake Service Registration

**Vector**: Attacker deploys a malicious ServiceNFT that reports false creator/pricing.
**Mitigation**: Only authorized factories can register in ServiceRegistry. `MaliciousServiceNFT` test validates this.

### Non-ERC721 Service

**Vector**: Attacker registers a contract that doesn't implement ERC-721.
**Mitigation**: ServiceFactory deploys ServiceNFT directly (not user-provided contracts). `MaliciousNonERC721` test validates interface checks.

### TBA Fund Theft

**Vector**: Executor moves TBA funds to unauthorized recipient.
**Mitigation**: `_isAllowedRecipient()` restricts executor transfers to exactly three addresses: Creator, FeeRecipient, Treasury.

### Signature Replay

**Vector**: Reuse of EIP-712 or EIP-3009 signatures.
**Mitigation**: Sequential nonces (EIP-712 in ServiceFactory) and random nonces (EIP-3009 in transfers). Deadlines prevent stale signatures.

### Front-Running

**Vector**: Miner/validator reorders transactions.
**Risk**: Low impact. Purchase prices are fixed (not AMM-based). Accepted risk.

## 5.5 Testing

The protocol has 1,000+ automated tests covering:

| Category | Coverage |
|----------|----------|
| Unit tests | Individual contract function testing |
| Integration tests | Multi-contract interaction flows |
| Security tests | Reentrancy, access control bypass, malicious contracts |
| Edge cases | Boundary values, overflow, dust amounts |
| End-to-end | Full lifecycle from registration to settlement |

Test contracts (`test/` directory):
- `MaliciousReentrantBuyer` --- reentrancy attack simulation
- `MaliciousServiceNFT` --- fake creator attack simulation
- `MaliciousNonERC721` --- interface bypass simulation
- `ServiceNFTBalanceHarness` --- internal function testing
- `ERC6551Registry` --- reference TBA registry for test deployments

## 5.6 Known Limitations & Planned Mitigations

| Limitation | Severity | Current State | Planned Mitigation |
|------------|----------|---------------|-------------------|
| No timelock on PlatformConfig updates | Medium | Admin can change addresses instantly | Add timelock + multi-sig |
| No contract upgrade path | Medium | Contracts are immutable | Evaluate proxy pattern for future versions |
| Centralized admin roles | Medium | Single admin address | Transition to multi-sig, then DAO |
| Executor is a single point of trust | Medium | One executor address | Decentralized executor network |
| No rate limiting on-chain | Low | Relies on gas costs | Off-chain rate limiting at relayer level |
# 6. Standards & Interoperability

## 6.1 Standards Summary

The Indie Protocol builds on 8 established Ethereum standards, ensuring maximum interoperability with the broader EVM ecosystem.

| Standard | Usage in Protocol | Contract |
|----------|-------------------|----------|
| **ERC-20** | inUSD stablecoin | InUSDToken |
| **ERC-721** | Service instance ownership | ServiceNFT |
| **ERC-1155** | Marketplace access passes | ServicePass |
| **ERC-5643** | Subscription billing | ServiceNFT |
| **ERC-6551** | Token-Bound Accounts (service wallets) | ServiceAccountImpl |
| **ERC-8004** | Machine-readable agent identity | ServiceNFT (agentId) |
| **EIP-712** | Typed structured data signing | ServiceFactory, InUSDToken |
| **EIP-3009** | Gasless token transfers | InUSDToken, TokenExchange, Marketplace |
| **EIP-4337** | Account abstraction (paymaster) | External (Coinbase CDP) |
| **x402** | HTTP 402 payment protocol | X402Receiver |

## 6.2 ERC-721: Service Instance Ownership

Each ServiceNFT instance is a standard ERC-721 token. This means:

- **Transferable**: Owners can transfer instances to other addresses
- **Composable**: Instances can be used in any ERC-721-compatible protocol (marketplaces, lending, fractionalization)
- **Enumerable**: ServiceNFT extends ERC721Enumerable for on-chain instance discovery
- **Metadata**: `tokenURI()` returns off-chain metadata URLs for service descriptions and images

### Composability Implications

Because service instances are standard NFTs, they can be:
- Listed on OpenSea, Blur, or any NFT marketplace for secondary trading
- Used as collateral in NFT lending protocols
- Bundled into collections
- Governed by DAO treasuries

## 6.3 ERC-5643: Subscription NFTs

ServiceNFT implements the ERC-5643 Subscription NFT standard, enabling:

```solidity
interface IERC5643 {
    function renewSubscription(uint256 tokenId, uint64 duration) external payable;
    function cancelSubscription(uint256 tokenId) external;
    function expiresAt(uint256 tokenId) external view returns (uint64);
    function isRenewable(uint256 tokenId) external view returns (bool);
    event SubscriptionUpdate(uint256 indexed tokenId, uint64 expiration);
}
```

This standard enables any external system to:
- Query subscription status without protocol-specific knowledge
- Build subscription management UIs that work across ERC-5643 tokens
- Integrate subscription logic into DeFi protocols (e.g., auto-renewal from yield)

## 6.4 ERC-6551: Token-Bound Accounts

The TBA standard gives each NFT instance its own smart contract wallet:

```
TBA Address = CREATE2(
    salt = bytes32(tokenId),
    implementation = ServiceAccountImpl,
    initCode = encode(chainId, serviceNFT, tokenId)
)
```

**Key properties:**
- **Deterministic**: TBA address is knowable before creation
- **Self-sovereign**: Owner controls the TBA (when unlocked)
- **Composable**: TBA can hold any ERC-20, ERC-721, or ERC-1155 tokens
- **Executable**: TBA can call any contract (when owner is authorized)
- **Signable**: TBA supports ERC-1271 smart contract signatures

### Why ERC-6551 for Services?

Traditional billing requires pre-approval of a spending contract with unlimited allowance --- a significant security risk. ERC-6551 inverts this:

| Traditional | Indie Protocol (ERC-6551) |
|-------------|---------------------------|
| User approves unlimited spending | User funds a dedicated wallet |
| Platform can drain approved amount | Platform can only transfer to 3 whitelisted addresses |
| Risk: unlimited exposure | Risk: limited to funded amount |
| User must revoke approvals | User withdraws remaining funds |

## 6.5 ERC-8004: Agent Identity

ServiceNFTs can link to an `agentId` via `setAgentId()`, implementing the ERC-8004 standard for machine-readable agent cards. This enables:

- **Agent-to-agent discovery**: Services can discover and compose with other services programmatically
- **AI integration**: LLM-based systems can read agent cards to understand service capabilities
- **Interoperability**: Standard agent identity format across protocols

## 6.6 EIP-712: Typed Data Signing

All gasless operations use EIP-712 typed structured data, providing:

- **Human-readable signing**: Users see exactly what they're authorizing in their wallet
- **Domain separation**: Signatures are bound to the specific contract and chain
- **Replay protection**: Nonces and deadlines prevent signature reuse

Used in:
- ServiceFactory: Service registration
- ERC20TransferWithAuthorization: Token transfers (EIP-3009)

## 6.7 EIP-3009: Transfer With Authorization

inUSD implements EIP-3009, enabling three gasless token operations:

```solidity
function transferWithAuthorization(
    address from, address to, uint256 value,
    uint256 validAfter, uint256 validBefore,
    bytes32 nonce, uint8 v, bytes32 r, bytes32 s
) external;

function receiveWithAuthorization(
    address from, address to, uint256 value,
    uint256 validAfter, uint256 validBefore,
    bytes32 nonce, uint8 v, bytes32 r, bytes32 s
) external;

function cancelAuthorization(
    address authorizer, bytes32 nonce,
    uint8 v, bytes32 r, bytes32 s
) external;
```

This standard is compatible with native USDC on L2s (Base, Arbitrum, Polygon), meaning the protocol's gasless UX pattern works with the most widely-used stablecoin infrastructure.

## 6.8 EIP-4337: Account Abstraction

The protocol leverages ERC-4337 paymasters (via Coinbase CDP) to sponsor gas for user transactions. Users never need to hold ETH:

- Frontend creates a UserOperation
- Paymaster signs the gas sponsorship
- Bundler submits the operation
- User's action executes without gas cost

This is an external integration, not part of the core protocol contracts.

## 6.9 x402: HTTP 402 Payment Protocol

X402Receiver enables services to monetize via the HTTP 402 "Payment Required" standard:

1. Client requests a paid resource
2. Server responds with HTTP 402 and payment details
3. Client pays via x402 facilitator (USDC)
4. X402Receiver collects the payment
5. Admin converts to inUSD via treasury

This bridges traditional HTTP APIs with the Indie Protocol's payment infrastructure, enabling services that are accessible via standard web requests to participate in the on-chain economy.

## 6.10 Interoperability Matrix

| External Protocol | Integration Point | How It Works |
|-------------------|-------------------|--------------|
| OpenSea / Blur | ServiceNFT (ERC-721) | Service instances tradeable on NFT marketplaces |
| Aave / Compound | ServiceAccountImpl (ERC-6551) | TBAs could hold yield-bearing assets |
| Gnosis Safe | PlatformConfig admin | Multi-sig governance for protocol admin |
| Uniswap / DEXs | InUSDToken (ERC-20) | inUSD tradeable on decentralized exchanges |
| Chainlink | Future integration | Price feeds for dynamic pricing models |
| IPFS / Arweave | Service metadata | Decentralized metadata storage option |
| Any ERC-4337 Paymaster | Transaction sponsorship | Gasless UX via any compatible paymaster |
# 7. Governance & Decentralization

## 7.1 Current State: Administered Protocol

The Indie Protocol currently operates under centralized administration. This is a deliberate choice for the bootstrapping phase, prioritizing rapid iteration, security response, and correct configuration.

### Current Admin Capabilities

| Capability | Contract | Risk Level |
|------------|----------|------------|
| Pause/unpause token transfers | InUSDToken | High (freezes economy) |
| Pause/unpause deposits/redemptions | InUSDCTreasury, TokenExchange | High |
| Update contract addresses | PlatformConfig | High (system-wide impact) |
| Set mint caps | InUSDToken | Medium |
| Authorize/deauthorize factories | ServiceRegistry | Medium |
| Grant/revoke relayer roles | Marketplace, ServiceFactory, TokenExchange | Medium |
| Set revenue beneficiary | InUSDCTreasury | Medium |
| Create new pass types | ServicePass | Low |
| Admin withdraw from treasury | InUSDCTreasury | High |
| Update metadata base URL | PlatformConfig | Low |

### Why Not a DAO from Day One?

1. **Security response time**: Smart contract bugs or exploits require immediate action (pause, reconfigure). DAO governance introduces delays that can cost millions.
2. **Configuration correctness**: Initial contract address configuration must be precise. One wrong address can break the entire system.
3. **Iteration speed**: Early-stage protocols need to evolve rapidly based on real usage data. Governance overhead slows this.
4. **Small stakeholder base**: With fewer than 1,000 users, a DAO is governance theater --- a few addresses would control all votes.

## 7.2 Decentralization Roadmap

The protocol will progressively decentralize through three phases:

### Phase 1: Multi-Signature (Current → Q3 2026)

**Objective**: Replace single admin address with multi-sig.

| Action | Implementation |
|--------|---------------|
| Transfer `DEFAULT_ADMIN_ROLE` to Gnosis Safe | 3-of-5 multi-sig |
| Transfer executor address to operational multi-sig | 2-of-3 for operational speed |
| Transfer revenue beneficiary to treasury multi-sig | 3-of-5 multi-sig |

**Impact**: No single person can unilaterally change protocol configuration. Compromise of one key does not compromise the protocol.

### Phase 2: Timelock (Q3 2026 → Q1 2027)

**Objective**: Add time delays to sensitive operations.

| Operation | Timelock Duration |
|-----------|-------------------|
| PlatformConfig address changes | 48 hours |
| Mint cap changes | 24 hours |
| Revenue beneficiary changes | 48 hours |
| Factory authorization changes | 24 hours |
| Emergency pause | Immediate (no timelock) |

**Impact**: Users have advance warning of protocol changes and can exit if they disagree. Emergency operations remain fast.

### Phase 3: Council Governance (Q1 2027 and beyond)

**Objective**: Expand decision-making to elected council.

- 7-member governance council (3 team, 2 creators, 2 community)
- Council votes on: fee changes, new pass types, factory authorizations
- Admin retains: emergency pause, security response
- Proposals require 5-of-7 approval + 24-hour timelock

### Future Governance Evolution

Further decentralization, including potential token-based governance, will be evaluated based on:
- Stable protocol operation with 6+ months of no critical incidents
- Demonstrated governance participation during the Council phase
- Regulatory clarity for on-chain governance structures
- Sufficient diversity of protocol stakeholders

The protocol prioritizes proven, incremental decentralization. No governance token is currently planned.

## 7.3 Immutability Considerations

### What Cannot Change

- ServiceNFT pricing is set at deployment and is immutable
- Service ownership follows ERC-721 rules (admin cannot seize NFTs)
- TBA authorization logic is hardcoded (admin cannot add new allowed recipients)
- BillingExecutor batch records are immutable once written

### What Can Change (With Governance)

- Contract addresses in PlatformConfig
- Mint caps on InUSDToken
- Revenue beneficiary address
- Factory authorization list
- ServicePass types and prices
- Metadata base URLs

### Upgrade Strategy

Current contracts are **not upgradeable** (no proxy pattern). This is intentional:
- Simpler security model (no proxy storage collision risks)
- Users can verify exact deployed bytecode
- No admin key can alter contract logic

For future versions, the protocol will deploy new contracts and migrate via PlatformConfig updates, with timelocks ensuring users can exit before changes take effect.

## 7.4 Economic Governance

### Fee Parameters

| Parameter | Current Value | Governance Control |
|-----------|---------------|-------------------|
| Exit fee | 10% | Hardcoded in treasury (immutable) |
| Execution fee | $0.0001 | Configurable by executor |
| Listing Pass price | Admin-set | ServicePass admin |
| Feature Pass price | Admin-set | ServicePass admin |
| Mint caps | $1M/tx, $10M/day | InUSDToken admin |

The 10% exit fee and 90% redemption rate are hardcoded in the InUSDCTreasury contract. Changing these requires deploying a new Treasury contract --- a deliberate design choice that provides strong guarantees to participants about the protocol's economic parameters.
# 8. Competitive Analysis & Market Positioning

## 8.1 Market Context

The Indie Protocol operates at the intersection of three converging markets:

| Market | Size (2025) | Growth | Relevance |
|--------|-------------|--------|-----------|
| Workflow Automation | $18B | 46% CAGR | Service execution and monetization |
| NFT/Digital Ownership | $2.5B | Recovering | Ownership primitives for digital services |
| DeFi/Payments | $170B TVL | Maturing | On-chain payment and billing rails |

No existing protocol addresses all three simultaneously. The Indie Protocol creates a new category: **service tokenization infrastructure**.

## 8.2 Competitive Positioning

### vs. Traditional Marketplaces

| Feature | Indie Protocol | AWS Marketplace | RapidAPI | Zapier |
|---------|---------------|-----------------|----------|--------|
| Commission on service revenue | 0% | 10-15% | 20% | 100% |
| IP ownership | On-chain (NFT) | Platform-controlled | Platform-controlled | Platform-controlled |
| Billing models | 3 (execution, subscription, purchase) | Per-use, subscription | Per-call | Per-seat subscription |
| Vendor lock-in | None (ERC-721 portable) | High (AWS ecosystem) | High (proprietary API) | High (proprietary format) |
| Secondary trading | Yes (any NFT marketplace) | No | No | No |
| Gasless UX | Yes | N/A (not crypto) | N/A | N/A |
| Composability | On-chain (service-to-service) | Limited (AWS only) | API chaining | Platform workflows |

### vs. Blockchain Service Protocols

| Feature | Indie Protocol | Gelato Network | Chainlink Functions | Akash Network |
|---------|---------------|----------------|--------------------| --------------|
| Service type | Any digital service | Blockchain automation | Decentralized compute | Compute marketplace |
| Billing model | Per-execution + subscription + purchase | Per-execution | Per-execution | Resource-based |
| NFT ownership | Yes (ERC-721) | No | No | No |
| TBA wallets | Yes (ERC-6551) | No | No | No |
| Stablecoin payments | Yes (inUSD/USDC) | ETH/MATIC | LINK | AKT |
| Gasless UX | Yes | Partial | No | No |
| Executor-agnostic | Yes | No (EVM-specific) | No (Chainlink DON) | Yes (containers) |

### vs. AI Agent Platforms

| Feature | Indie Protocol | OpenAI GPT Store | CrewAI | AutoGPT |
|---------|---------------|-----------------|--------|---------|
| Commission on service revenue | 0% | ~30% | N/A | N/A |
| IP ownership | On-chain NFT | OpenAI controls | Self-hosted | Self-hosted |
| Agent types | Any service | Chat agents only | AI crews | AI agents |
| Billing | Flexible (3 models) | Subscription-based | None | None |
| Composability | On-chain service-to-service | Closed ecosystem | Code-level | Code-level |
| Monetization infra | Built-in | Built-in | None | None |

## 8.3 Unique Differentiators

### 1. Executor-Agnostic Design

The protocol does not mandate how services run. This is its most fundamental differentiator. While Gelato requires EVM transactions, Chainlink requires their DON, and Akash requires containers --- the Indie Protocol works with **any** execution backend that can report billing events.

### 2. Three-Model Billing in One Framework

No competing protocol offers per-execution, subscription, and one-time purchase in a single smart contract framework. Traditional marketplaces support one model; the Indie Protocol supports all three simultaneously per service.

### 3. Token-Bound Account Billing

The ERC-6551 "fund once, execute many" model is unique to the Indie Protocol. Users fund a dedicated wallet rather than approving unlimited spending --- a fundamentally safer billing model.

### 4. NFT-Based Service Ownership

Service instances are standard ERC-721 tokens, enabling secondary trading, collateralization, DAO governance, and cross-protocol composability. No competitor offers this.

### 5. Gasless Complete UX

All user-facing operations are gasless: registration, listing, purchasing, and swapping. This removes the crypto adoption barrier while preserving on-chain security guarantees.

## 8.4 Target Use Cases

The protocol is designed for platforms and builders in:

| Use Case | Value Proposition |
|----------|-------------------|
| **AI Agent Marketplaces** | Plug-and-play ownership, billing, and marketplace for AI agents |
| **API Monetization** | Per-call billing via TBAs, no API key management needed |
| **Workflow Automation** | Mint workflows as NFTs, earn per-execution revenue |
| **Compute Marketplaces** | Metered billing for GPU/CPU tasks via TBAs |
| **SaaS Micro-Services** | Subscription billing with on-chain ownership |
| **Data Pipeline Markets** | Subscription feeds with verifiable delivery via BillingExecutor |
| **Consulting Platforms** | One-time purchase model for professional deliverables |

## 8.5 Moat and Defensibility

### Network Effects

Two-sided marketplace dynamics create compounding value:
- More services listed → more buyers attracted → more revenue for creators → more creators
- Service composition (service-to-service calls) increases with catalog size

### Standard Adoption

Building on established ERCs means the protocol benefits from growing ecosystem support:
- ERC-6551 adoption increases TBA tooling and integrations
- ERC-5643 adoption increases subscription management infrastructure
- ERC-721 adoption means instant compatibility with all NFT infrastructure

### Data Moat

BillingExecutor accumulates on-chain reputation data (execution counts, revenue, reliability) that cannot be replicated by competitors. This data becomes increasingly valuable for buyer decision-making.

### Switching Costs

Once a platform integrates the Indie Protocol's smart contracts, switching requires:
- Migrating all NFT ownership records
- Migrating all TBA balances
- Rebuilding billing infrastructure
- Losing on-chain reputation data
# 9. Roadmap & Milestones

## 9.1 Current Status

The Indie Protocol smart contracts are deployed on **Base Sepolia** (testnet) with 1,000+ automated tests passing. All 12 production contracts are functional and integrated.

### Shipped Capabilities

| Capability | Status | Details |
|------------|--------|---------|
| Service tokenization (ERC-721) | Shipped | ServiceNFT with immutable pricing |
| Subscription billing (ERC-5643) | Shipped | 30-day recurring with auto-cancel on transfer |
| Token-Bound Accounts (ERC-6551) | Shipped | Dual-auth model with fund-and-execute |
| Gasless marketplace (EIP-712/3009) | Shipped | Register, list, purchase, swap --- all gasless |
| inUSD stablecoin | Shipped | 1:1 USDC backing, mint caps, pausable |
| ServicePass (ERC-1155) | Shipped | Listing and Feature passes |
| Batch settlement | Shipped | BillingExecutor with audit trail |
| Agent identity (ERC-8004) | Shipped | Machine-readable agent cards |
| x402 payment support | Shipped | HTTP 402 USDC settlement |
| ERC-4337 paymaster (gasless) | Shipped | Via Coinbase CDP integration |
| Multi-chain testnet support | Shipped | Base Sepolia + Arbitrum Sepolia |

## 9.2 Roadmap

### Phase 1: Foundation (Q1 2026) --- Current

**Objective**: Complete smart contract framework, security review, and first executor integration.

| Milestone | Target | Status |
|-----------|--------|--------|
| 12 production contracts deployed on testnet | Q1 2026 | Done |
| 1,000+ automated tests | Q1 2026 | Done |
| First executor integration (proof of concept) | Q1 2026 | Done |
| Protocol website and whitepaper | Q1 2026 | In Progress |
| Security audit (internal) | Q1 2026 | In Progress |

### Phase 2: Mainnet Launch (Q2 2026)

**Objective**: Deploy to Base Mainnet, onboard first creators, secure initial funding.

| Milestone | Target |
|-----------|--------|
| External security audit | Q2 2026 |
| Base Mainnet deployment | Q2 2026 |
| Multi-sig governance (Phase 1 governance) | Q2 2026 |
| 10 services listed | Q2 2026 |
| $10K protocol TVL | Q2 2026 |
| Grant applications (Base, Chainlink, ChainGPT) | Q2 2026 |

### Phase 3: Growth (Q3-Q4 2026)

**Objective**: Scale supply and demand, prove unit economics, expand to additional chains.

*Conditional on at least 2 active executor integrations by Q3 2026.*

| Milestone | Target |
|-----------|--------|
| Arbitrum Mainnet deployment | Q3 2026 |
| Timelock governance (Phase 2 governance) | Q3 2026 |
| SDK and documentation for executor integrations | Q3 2026 |
| Second executor integration | Q4 2026 |

**Aspirational KPIs** (not hard milestones):

| KPI | Q4 2026 Target |
|-----|----------------|
| Services listed | 100 |
| Active buyers | 1,000 |
| Daily executions | 10,000 |

### Phase 4: Protocol Maturity (2027)

**Objective**: Decentralize governance, expand chain support, enable advanced features.

| Milestone | Target |
|-----------|--------|
| Council governance (Phase 3 governance) | Q1 2027 |
| Dynamic pricing model (oracle-based) | Q2 2027 |
| Service reputation system (on-chain) | Q2 2027 |
| Cross-chain service discovery | Q3 2027 |
| Executor SDK v2 with standard interface | Q3 2027 |
| 5+ executor integrations | Q4 2027 |

### Future Considerations

Further decentralization, including potential token-based governance, will be evaluated based on protocol maturity, demonstrated governance participation during the Council phase, and regulatory clarity. The protocol prioritizes proven, incremental decentralization over speculative governance structures.

## 9.3 Funding Strategy

### Grant Targets

| Tier | Grant Program | Amount | Focus |
|------|--------------|--------|-------|
| **Tier 1** (Fast Track) | Base Builder Grants | 1-5 ETH | Ecosystem development |
| **Tier 1** | ChainGPT Web3 AI Grant | From $1M pool | AI service infrastructure |
| **Tier 1** | Chainlink Community Grant | $100K+ | Oracle integration |
| **Tier 2** (Deep Tech) | ASI Alliance Deep Funding | Variable | AI agent economics |
| **Tier 2** | Arbitrum DAO Grants | $150K | After $1M GMV milestone |
| **Tier 3** (Long Game) | Uniswap Grants | Variable | DeFi integration |

### Revenue Projections

Revenue projections are scenario-based and depend on executor adoption --- the primary growth constraint. The protocol currently has one executor integration (proof of concept). All figures assume mainnet launch in Q2 2026.

#### Conservative Scenario (1 executor, early traction)

| Metric | Q4 2026 | Q4 2027 | Q4 2028 |
|--------|---------|---------|---------|
| Daily Executions | 500 | 5K | 25K |
| Monthly GMV | $1.5K | $15K | $75K |
| Exit Fee Revenue (10%) | $150/mo | $1.5K/mo | $7.5K/mo |
| Execution Fee Revenue | $1.50/mo | $15/mo | $75/mo |
| Pass Sales Revenue | $300/mo | $1.5K/mo | $5K/mo |
| **Total Protocol Revenue** | **$500/mo** | **$3K/mo** | **$12.5K/mo** |

#### Base Scenario (2 executors, moderate growth)

| Metric | Q4 2026 | Q4 2027 | Q4 2028 |
|--------|---------|---------|---------|
| Daily Executions | 2K | 25K | 150K |
| Monthly GMV | $7K | $50K | $200K |
| Exit Fee Revenue (10%) | $700/mo | $5K/mo | $20K/mo |
| Execution Fee Revenue | $6/mo | $75/mo | $450/mo |
| Pass Sales Revenue | $800/mo | $3K/mo | $10K/mo |
| **Total Protocol Revenue** | **$1.5K/mo** | **$8K/mo** | **$30K/mo** |

#### Optimistic Scenario (3+ executors, strong adoption)

| Metric | Q4 2026 | Q4 2027 | Q4 2028 |
|--------|---------|---------|---------|
| Daily Executions | 10K | 100K | 1M |
| Monthly GMV | $25K | $100K | $500K |
| Exit Fee Revenue (10%) | $2.5K/mo | $10K/mo | $50K/mo |
| Execution Fee Revenue | $30/mo | $300/mo | $3K/mo |
| Pass Sales Revenue | $1K/mo | $5K/mo | $20K/mo |
| **Total Protocol Revenue** | **$3.5K/mo** | **$15K/mo** | **$73K/mo** |

> **Note**: The Conservative scenario reflects a single executor with limited marketplace activity. The Optimistic scenario requires multiple successful executor integrations and organic demand growth. Actual results depend on executor partner adoption.

## 9.4 Key Performance Indicators

| KPI | Definition | Q4 2026 Target (Base) |
|-----|-----------|----------------|
| **Executor Integrations** | Active executor partners | 2 |
| **Services Listed** | Active, non-paused services | 50 |
| **Active Buyers** | Addresses with 1+ execution in 30 days | 500 |
| **Daily Executions** | Total service executions per day | 2,000 |
| **TVL** | inUSD supply + TBA balances | $25K |
| **Creator Retention** | % of creators with 2+ services | 30% |
| **Buyer Retention** | % of buyers with 30-day activity | 40% |
# 10. Risk Factors

## 10.1 Technical Risks

### Smart Contract Risk

| Risk | Severity | Mitigation |
|------|----------|------------|
| Undiscovered vulnerability in contracts | High | 1,000+ tests, internal audit, planned external audit |
| No contract upgrade path (immutable) | Medium | Deliberate design; new versions deploy fresh with PlatformConfig migration |
| OpenZeppelin dependency vulnerability | Low | Pinned to v5, monitoring for upstream CVEs |

### Executor Centralization

The executor is a trusted off-chain component. If compromised, it could:
- Move TBA funds to whitelisted recipients improperly (billing fraud)
- Record false statistics in BillingExecutor
- Lock/unlock instances without legitimate execution

**Mitigation**: Executor transfers are restricted to three whitelisted addresses. BillingExecutor creates an auditable trail. Progressive decentralization through multi-sig, timelock, and council governance reduces single-point-of-failure risk.

### Platform Configuration Risk

PlatformConfig admin can change any contract address, effectively redirecting the entire protocol. This is the highest-severity admin risk.

**Mitigation**: Multi-sig governance (Phase 1), timelock (Phase 2), council governance (Phase 3).

## 10.2 Economic Risks

### Insufficient Liquidity

If inUSD supply is low, the ecosystem cannot support high-value services. Buyers cannot fund TBAs; creators cannot receive meaningful payments.

**Mitigation**: Free entry (0% deposit fee) minimizes friction. Protocol operations (pass purchases, execution fees) create baseline demand for inUSD.

### Exit Fee Sensitivity

The 10% exit fee may deter participation if:
- Participants primarily want short-term interactions
- Competing protocols offer lower extraction
- The fee is perceived as excessive

**Mitigation**: The fee is only charged on exit. Participants who spend inUSD within the ecosystem pay 0%. Market testing will inform whether adjustment is needed (requires new Treasury deployment).

### USDC Dependency

inUSD is 100% backed by USDC. A USDC depeg event would propagate to inUSD.

**Mitigation**: USDC is the most widely-used regulated stablecoin with $50B+ in circulation. The risk of a full depeg is low but non-zero. Treasury reserves are held in USDC only (no exotic yield strategies).

## 10.3 Market Risks

### Adoption Failure

The protocol requires both supply (creators listing services) and demand (buyers purchasing instances). Classic cold-start problem.

**Mitigation**: Initial focus on a single executor integration to demonstrate end-to-end value. Creator onboarding programs via grants and partnerships.

### Competition

Established platforms (AWS, RapidAPI, Zapier) could add similar features. Blockchain-native protocols (Gelato, Chainlink) could expand their scope.

**Mitigation**: The protocol's executor-agnostic design means it competes as infrastructure, not as a platform. Multiple executor integrations create diversified demand.

### Regulatory Risk

Depending on jurisdiction, aspects of the protocol may face regulatory scrutiny:
- inUSD could be classified as a payment instrument or e-money
- NFT service instances could be classified as securities (if speculative trading is prominent)
- Cross-border service payments may trigger compliance requirements

**Mitigation**: Legal analysis of stablecoin regulations in target jurisdictions. Conservative marketing that emphasizes utility over speculation.

## 10.4 Operational Risks

### Key Person Risk

Early-stage projects depend on a small team. Loss of key contributors could stall development.

**Mitigation**: Comprehensive documentation, automated testing, and clean code practices reduce bus factor. Grant funding enables team expansion.

### Relayer Availability

The gasless UX depends on relayer uptime. If the relayer goes down, users cannot perform gasless operations (they can still interact directly, but UX degrades).

**Mitigation**: Relayer infrastructure on redundant cloud providers. Direct contract interaction as fallback.

### Off-Chain Storage

Service metadata (names, descriptions, workflow definitions) is stored off-chain. Loss of this data does not affect ownership or billing (which are on-chain) but would degrade user experience.

**Mitigation**: Multiple storage backends (KV + R2). Planned IPFS/Arweave integration for permanent storage of critical metadata.

## 10.5 Risk Summary Matrix

| Risk | Probability | Impact | Mitigation Priority |
|------|-------------|--------|---------------------|
| Smart contract vulnerability | Low | Critical | External audit (Q2 2026) |
| Executor compromise | Low | High | Multi-sig + audit trail |
| PlatformConfig admin abuse | Low | Critical | Multi-sig + timelock |
| Insufficient liquidity | Medium | High | Free entry + partnerships |
| Exit fee deterrence | Medium | Medium | Market testing + adjustment |
| USDC depeg | Very Low | Critical | Accept + monitor |
| Adoption failure (cold start) | Medium | High | Focused executor integration |
| Competition response | Medium | Medium | Executor-agnostic positioning |
| Regulatory action | Low | High | Legal analysis + compliance |
| Relayer downtime | Low | Medium | Redundancy + direct fallback |
# Appendix A: Smart Contract Reference

## A.1 Contract Deployment Addresses

*Addresses will be published upon mainnet deployment. Current testnet deployments are on Base Sepolia (chain ID 84532).*

## A.2 Contract Specifications

### InUSDToken

| Property | Value |
|----------|-------|
| Standard | ERC-20 + EIP-3009 |
| Decimals | 6 |
| Symbol | inUSD |
| Max Mint/Tx | 1,000,000 × 10^6 |
| Max Mint/Day | 10,000,000 × 10^6 |
| Backing Requirement | 90% USDC at mint time |
| Roles | DEFAULT_ADMIN, MINTER, TREASURY |
| Pausable | Yes |
| Burnable | Yes (ERC20Burnable) |

### InUSDCTreasury

| Property | Value |
|----------|-------|
| Redemption Rate | 9000 (90% of 10000 basis points) |
| Min Withdrawal | 1 × 10^6 ($1) |
| Revenue Split | 10% to revenueBeneficiary on deposit |
| Roles | DEFAULT_ADMIN, EXCHANGER |
| Guards | ReentrancyGuard, Pausable |

### TokenExchange

| Property | Value |
|----------|-------|
| Entry Rate | 1:1 (USDC → inUSD) |
| Exit Rate | 90% (inUSD → USDC) |
| Exit Fee | 10% (1000 basis points) |
| Min Amount | 1 × 10^6 ($1) |
| Max Amount | 500,000 × 10^6 ($500K) |
| Roles | DEFAULT_ADMIN, RELAYER |
| Guards | ReentrancyGuard, Pausable |

### ServiceFactory

| Property | Value |
|----------|-------|
| Signing Standard | EIP-712 |
| Nonce Type | Sequential per creator |
| Roles | DEFAULT_ADMIN, RELAYER |
| Guards | ReentrancyGuard |

### ServiceNFT

| Property | Value |
|----------|-------|
| Standard | ERC-721 + ERC-721Enumerable + ERC-5643 |
| Subscription Period | 30 days |
| Pricing | Immutable at deployment |
| Token IDs | Auto-incrementing from 1 |
| Pausable | Yes (by creator or marketplace) |
| Lockable | Yes (lock/unlock by owner and executor) |
| Agent Identity | ERC-8004 agentId |
| Guards | ReentrancyGuard (renewSubscription) |

### ServiceRegistry

| Property | Value |
|----------|-------|
| Factory Authorization | Whitelist-based |
| Ownership Tracking | Via EnumerableSet per user |
| Pagination | Supported (offset + limit) |
| Roles | DEFAULT_ADMIN |

### Marketplace

| Property | Value |
|----------|-------|
| Listing Pass ID | 0 |
| Feature Pass ID | 1 |
| Max Boost Duration | 30 days |
| Analytics | CQRS (per-creator, per-buyer) |
| Roles | DEFAULT_ADMIN, RELAYER |
| Guards | ReentrancyGuard |

### BillingExecutor

| Property | Value |
|----------|-------|
| Max Batch Size | 250 |
| Batch IDs | Sequential |
| Stats | Per-creator, per-service aggregates |
| Authorization | onlyExecutor (from PlatformConfig) |
| Guards | ReentrancyGuard |

### ServiceAccountImpl (ERC-6551 TBA)

| Property | Value |
|----------|-------|
| Standard | ERC-6551 + ERC-1271 |
| Auth: Locked state | Executor only → transfers to whitelisted recipients |
| Auth: Unlocked state | NFT Owner only → any transaction |
| Batch Support | batchExecute(calls[]) |
| Signature Support | ERC-1271 (unlocked only) |

### ServicePass

| Property | Value |
|----------|-------|
| Standard | ERC-1155 |
| Pass ID 0 | Listing Pass |
| Pass ID 1 | Feature Pass |
| Extensible | New pass types via createPassType() |
| Roles | DEFAULT_ADMIN, BURNER |

### X402Receiver

| Property | Value |
|----------|-------|
| Token | USDC only |
| Roles | DEFAULT_ADMIN |
| Guards | ReentrancyGuard |

### PlatformConfig

| Property | Value |
|----------|-------|
| Config Keys | 12 (EXECUTOR through SERVICE_PASS) |
| Min Execution Price | 10,000 ($0.01) |
| Roles | DEFAULT_ADMIN |
| Immutable | inusdToken address |

## A.3 Event Reference

### InUSDToken Events
- `TokensMinted(address to, uint256 amount)`
- `TokensBurned(address from, uint256 amount)`
- `TreasuryConfigured(address treasury, address usdtToken)`
- `MaxMintPerTransactionUpdated(uint256 newMax)`
- `MaxDailyMintUpdated(uint256 newMax)`

### InUSDCTreasury Events
- `USDCDeposited(address depositor, address recipient, uint256 amount)`
- `InUSDBurned(address redeemer, uint256 inusdAmount, uint256 usdcAmount)`
- `RevenueDistributed(address beneficiary, uint256 amount)`
- `RevenueBeneficiaryUpdated(address newBeneficiary)`
- `AdminWithdrawal(address to, uint256 amount)`

### Marketplace Events
- `InstancePurchased(address buyer, address service, uint256 tokenId, uint256 price)`
- `ServiceListed(address service)`
- `ServiceBoosted(address service, uint256 durationDays)`

### ServiceNFT Events
- `InstanceMinted(uint256 tokenId, address buyer)`
- `ServicePaused(bool paused)`
- `InstanceLockUpdated(uint256 tokenId, bool locked)`
- `AgentIdUpdated(uint256 agentId)`
- `SubscriptionUpdate(uint256 tokenId, uint64 expiration)` (ERC-5643)

### ServiceFactory Events
- `ServiceRegistered(address service, address creator, bytes32 serviceHash)`
- `ServiceDeployed(address service, address creator)`

### BillingExecutor Events
- `Settled(uint256 batchId, uint256 totalAmount, uint256 executionCount)`

### ServiceRegistry Events
- `ProductRegistered(address product, address creator)`
- `FactoryAuthorized(address factory, bool authorized)`
- `OwnershipAdded(address product, address user)`
- `OwnershipRemoved(address product, address user)`

### TokenExchange Events
- `Swap(address user, bool isEntry, uint256 amountIn, uint256 amountOut)`

## A.4 Compiler & Dependencies

| Component | Version |
|-----------|---------|
| Solidity | ^0.8.20 |
| OpenZeppelin Contracts | v5.x |
| Hardhat | 2.x |
| Optimizer | Enabled (runs=1, optimize for size) |
| Target Chain | Base (EVM London fork) |
# Appendix B: Glossary

| Term | Definition |
|------|-----------|
| **Agent** | A digital service deployed through the Indie Protocol, represented as a ServiceNFT contract |
| **Builder / Creator** | A developer who deploys services on the protocol and earns revenue from execution, purchase, and subscription payments |
| **Buyer / Producer** | A user who purchases service instances and funds them for execution |
| **Circuit Breaker** | The `maxExecutionPrice` parameter on ServiceNFT that halts execution if projected cost exceeds the ceiling |
| **CQRS** | Command Query Responsibility Segregation --- the architectural pattern separating on-chain writes from off-chain reads |
| **EIP-3009** | Ethereum standard for gasless token transfers via off-chain signed authorizations |
| **EIP-712** | Ethereum standard for typed structured data signing, enabling human-readable transaction authorization |
| **ERC-721** | Ethereum standard for non-fungible tokens (NFTs), used for service instance ownership |
| **ERC-1155** | Ethereum standard for multi-token contracts, used for ServicePass access tokens |
| **ERC-4337** | Ethereum account abstraction standard enabling gasless transactions via paymasters |
| **ERC-5643** | Ethereum standard for subscription NFTs, enabling on-chain recurring billing |
| **ERC-6551** | Ethereum standard for Token-Bound Accounts (TBAs) --- smart contract wallets owned by NFTs |
| **ERC-8004** | Ethereum standard for machine-readable agent identity cards |
| **Execution Price** | Per-run cost set by the creator, deducted from the instance's TBA on each execution |
| **Executor** | The authorized off-chain system that runs services, manages billing, and records settlement data |
| **Exit Fee** | The 10% fee charged when converting inUSD back to USDC |
| **Feature Pass** | ServicePass (ID 1) burned to boost a service into the marketplace's featured section |
| **Fund Once, Execute Many** | The billing model where users deposit inUSD into a TBA and executions draw from that balance |
| **Gasless** | Operations that require no ETH from the user; gas is sponsored by a relayer or paymaster |
| **inUSD** | The protocol's native stablecoin, pegged 1:1 to USDC, used as the unit of account for all operations |
| **Instance** | A specific token (ERC-721) representing a user's license/copy of a service |
| **Listing Pass** | ServicePass (ID 0) burned to list a service on the marketplace |
| **Locked** | Instance state where only the executor can move TBA funds (to whitelisted recipients) |
| **Max Execution Price** | Circuit breaker ceiling; execution halts if projected cost exceeds this value |
| **Paymaster** | An ERC-4337 component that sponsors gas fees for user transactions |
| **PlatformConfig** | The central on-chain registry storing all protocol contract addresses |
| **Purchase Price** | One-time cost to acquire a service instance, paid in inUSD from buyer to creator |
| **Redemption Rate** | The rate at which inUSD can be converted back to USDC (90%, or 9000/10000 basis points) |
| **Relayer** | A backend service that submits gasless transactions on behalf of users |
| **Revenue Beneficiary** | Address receiving the 10% USDC revenue from inUSD exit fees |
| **Service** | A digital offering (API, agent, workflow, compute task) deployed as a ServiceNFT contract |
| **ServiceNFT** | An ERC-721 contract representing a service; token IDs are instances of that service |
| **ServicePass** | An ERC-1155 contract whose tokens gate marketplace actions (listing, boosting) |
| **Subscription Price** | 30-day recurring cost for unlimited executions, paid in inUSD from buyer to creator |
| **TBA** | Token-Bound Account --- an ERC-6551 smart contract wallet owned by and derived from an NFT |
| **Treasury** | The InUSDCTreasury contract that holds USDC reserves backing inUSD |
| **Unlocked** | Instance state where the NFT owner has full control of the TBA |
| **x402** | HTTP 402 "Payment Required" protocol enabling web-native service payments in USDC |
