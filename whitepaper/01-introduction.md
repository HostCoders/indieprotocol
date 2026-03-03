# 1. Introduction

## 1.1 The Service Economy Problem

Digital services --- APIs, AI agents, automation workflows, compute tasks, data pipelines, consulting deliverables --- form the backbone of the modern economy. Yet the infrastructure for owning, trading, and billing these services remains fragmented and centralized:

- **Ownership is platform-dependent.** A developer's API, agent, or workflow exists at the pleasure of the platform hosting it. There is no portable, verifiable ownership primitive for digital services.
- **Billing is rigid.** Most platforms force a single model: monthly subscriptions, per-seat licensing, or flat-rate APIs. There is no unified framework that supports per-execution micropayments, subscriptions, and one-time purchases simultaneously.
- **Discovery is siloed.** Services live in walled-garden marketplaces (AWS Marketplace, Zapier, RapidAPI) with platform-specific listing, purchasing, and settlement flows.
- **Composability is absent.** Service A cannot trustlessly call and pay Service B without both being on the same platform with the same billing system.

The Indie Protocol addresses these structural gaps with a general-purpose smart contract framework that any service --- regardless of its execution environment --- can use for tokenization, marketplace access, and on-chain billing.

## 1.2 What the Indie Protocol Is

The Indie Protocol is a **set of composable smart contracts** deployed on EVM-compatible chains that provide:

1. **Service Tokenization**: Mint any digital service as an ERC-721 NFT with built-in subscription support (ERC-5643). Each instance of the service is a token. Ownership is transferable, verifiable, and composable.

2. **Decentralized Marketplace**: List, discover, purchase, and boost services through an on-chain marketplace with gasless UX. No intermediary controls listing or delisting.

3. **Flexible Billing**: Three billing models in one framework --- per-execution (metered), subscription (30-day recurring), and one-time purchase --- all settled in inUSD.

4. **Token-Bound Accounts**: Each service instance gets a dedicated ERC-6551 wallet. Users fund the wallet; the executor draws from it per execution. Full owner control when idle.

5. **Payment Infrastructure**: inUSD stablecoin (1:1 USDC-backed) with gasless swaps, providing a stable unit of account for all protocol operations.

6. **Access Gating**: ERC-1155 ServicePass tokens gate marketplace actions (listing, boosting), creating a spam-resistant, fee-based access model.

## 1.3 What the Indie Protocol Is Not

The protocol explicitly does **not** prescribe:

- **How services execute.** The protocol is executor-agnostic. An AI agent platform, a workflow engine, an API gateway, or a human services marketplace can all plug into the same contracts.
- **Where services run.** Services can run on Cloudflare Workers, AWS Lambda, bare metal, or a developer's laptop. The protocol handles ownership and billing; execution is external.
- **What services do.** From LLM inference to data transformation to image generation to legal document review --- any digital service that can report execution events can use the protocol.

This separation of concerns is fundamental to the protocol's design. The smart contracts define the **economic layer** for services; execution environments are pluggable implementations.

## 1.4 Design Principles

| Principle | Description |
|-----------|-------------|
| **Executor-Agnostic** | The protocol defines ownership, billing, and marketplace. Execution is pluggable. Any backend can integrate. |
| **Creator-First Economics** | Service creators keep 100% of execution and purchase revenue. The protocol takes zero commission on service sales. |
| **Gasless by Default** | All user-facing operations (register, list, purchase, swap) are gasless via EIP-712, EIP-3009, and ERC-4337 paymasters. |
| **Composable Primitives** | Each contract is independently useful. Use ServiceNFT without the Marketplace. Use inUSD without ServiceNFTs. Mix and match. |
| **Minimal Extraction** | Protocol sustainability through a 10% exit fee on USDC withdrawals and flat per-execution fees --- no rent-seeking on creator revenue. |
| **Standards-First** | Built on established ERCs (721, 1155, 5643, 6551, 8004) and EIPs (712, 3009, 4337) for maximum interoperability. |

## 1.5 Example Integrations

To illustrate the protocol's generality, here are potential executor implementations:

| Executor Type | How It Uses the Protocol |
|---------------|--------------------------|
| **AI Agent Platform** | Each agent is a ServiceNFT. Buyers purchase instances, fund TBAs, and the platform executor bills per inference call. |
| **Workflow Automation Engine** | Each workflow template is a ServiceNFT. Buyers run workflows; per-execution billing deducts from the TBA. |
| **API Marketplace** | Each API endpoint is a ServiceNFT. Consumers purchase access; metered billing via TBA or subscription via ERC-5643. |
| **Compute Marketplace** | Each compute task type is a ServiceNFT. Jobs are billed per execution from the TBA. |
| **Consulting/Freelance Platform** | Each service offering is a ServiceNFT. One-time purchase model for deliverables. |
| **Data Pipeline Marketplace** | Each ETL pipeline is a ServiceNFT. Subscription model for continuous data feeds. |

## 1.6 Document Structure

This whitepaper is organized as follows:

| Section | Content |
|---------|---------|
| **2** | Protocol architecture and smart contract design |
| **3** | Token economics and fee structure (inUSD) |
| **4** | Service lifecycle: registration, listing, purchase, execution, settlement |
| **5** | Security model, access control, and trust assumptions |
| **6** | Standards and interoperability (ERCs, EIPs, x402) |
| **7** | Governance and decentralization roadmap |
| **8** | Competitive analysis and market positioning |
| **9** | Roadmap and milestones |
| **10** | Risk factors |
| **A** | Appendix: Smart contract reference |
| **B** | Appendix: Glossary |
