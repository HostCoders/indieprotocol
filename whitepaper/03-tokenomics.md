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
