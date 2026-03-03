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
