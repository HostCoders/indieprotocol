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
