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
