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
