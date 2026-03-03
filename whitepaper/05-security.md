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
