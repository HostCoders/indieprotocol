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
