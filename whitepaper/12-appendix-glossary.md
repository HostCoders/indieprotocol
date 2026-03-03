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
