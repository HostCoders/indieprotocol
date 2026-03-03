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
