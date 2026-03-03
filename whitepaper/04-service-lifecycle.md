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
