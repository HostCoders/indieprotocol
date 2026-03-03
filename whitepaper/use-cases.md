# Indie Protocol: Use Cases

## How the Protocol Primitives Map to Real-World Applications

The Indie Protocol provides six core primitives. Every use case below is a combination of these:

| Primitive | What It Enables |
|-----------|----------------|
| **ServiceNFT** (ERC-721) | Tokenize any digital service as a transferable, ownable asset |
| **Token-Bound Account** (ERC-6551) | Dedicated wallet per service instance: fund once, execute many |
| **Three Billing Models** | Per-execution, 30-day subscription, one-time purchase --- in one contract |
| **Gasless Marketplace** | List, discover, purchase, boost --- zero gas for end users |
| **inUSD Payment Rails** | Stable, USDC-backed currency with free entry and ecosystem retention |
| **BillingExecutor** | On-chain audit trail for batch settlement, reputation, and analytics |

---

## 1. AI Agent Marketplace

### The Problem
AI agent builders (using frameworks like LangChain, CrewAI, AutoGen, or custom pipelines) have no standard way to package, sell, and bill for their agents. Agents live in GitHub repos or private deployments with no monetization path.

### How the Indie Protocol Solves It

- **Each AI agent is a ServiceNFT.** The builder deploys a ServiceNFT with an `executionPrice` (e.g., $0.05 per inference) and optionally a `subscriptionPrice` (e.g., $20/month for unlimited calls).
- **Buyers purchase instances.** A business purchases an instance NFT, funds the TBA with $50 in inUSD, and the agent platform draws $0.05 per call from the TBA.
- **Circuit breaker protects buyers.** `maxExecutionPrice` halts execution if a single call would exceed, say, $2.00 --- preventing runaway LLM costs (GPT-4 long-context, chain-of-thought loops).
- **Agent identity (ERC-8004)** enables agent-to-agent discovery. Agent A can find and call Agent B on-chain, with billing handled transparently between TBAs.
- **Reputation accrues on-chain.** BillingExecutor records execution count and revenue per agent, building a verifiable track record.

### Billing Model
Per-execution (primary) + Subscription (power users)

### Example Flow
```
Builder deploys "Legal Contract Reviewer" agent
    → executionPrice: $0.25 per document
    → subscriptionPrice: $50/month unlimited
    → maxExecutionPrice: $5.00

Buyer (law firm) purchases instance for $100 (purchasePrice)
    → Funds TBA with $200 inUSD
    → Submits 400 documents over 2 months
    → TBA deducts $0.25 per doc → Builder earns $100
    → Buyer refunds TBA to continue
```

---

## 2. API Monetization Platform

### The Problem
Independent developers build valuable APIs (data enrichment, geocoding, sentiment analysis, image processing) but monetizing them requires building billing infrastructure, managing API keys, handling payment disputes, and tracking usage --- all undifferentiated work.

### How the Indie Protocol Solves It

- **Each API endpoint (or API product) is a ServiceNFT.** The developer sets a `purchasePrice` for access and an `executionPrice` per API call.
- **TBA replaces API key management.** Instead of issuing API keys and tracking usage in a database, the protocol handles access (you own the NFT = you have access) and billing (TBA balance = your credit).
- **No payment processor needed.** No Stripe integration, no invoice generation, no chargebacks. inUSD payments are final and on-chain.
- **x402 integration** enables standard HTTP 402 responses. The API can serve paid content via native web standards --- no custom auth headers needed.

### Billing Model
Per-execution (metered API calls) + One-time purchase (premium data sets)

### Example Flow
```
Developer deploys "Company Enrichment API"
    → executionPrice: $0.01 per lookup
    → purchasePrice: $25 (one-time access)

Buyer (sales team) purchases instance
    → Funds TBA with $100 inUSD
    → Makes 5,000 API calls over 3 months
    → TBA deducted $50 → Developer earns $50
    → Buyer still has $50 remaining in TBA
```

---

## 3. Workflow Automation Marketplace

### The Problem
Workflow automation builders (n8n, Make, Zapier-style) create valuable multi-step automations connecting APIs, transforming data, and orchestrating business processes. There is no marketplace to sell these as products with recurring billing.

### How the Indie Protocol Solves It

- **Each workflow template is a ServiceNFT.** The builder sets pricing based on the workflow's value and compute cost.
- **Buyers own their instance.** Unlike SaaS subscriptions where you lose access when you stop paying, the NFT is owned permanently. Subscription is only for unlimited execution, not for ownership.
- **Workflow-to-workflow composability.** One workflow can call another. Each call is independently billed from the caller's TBA, enabling a composable automation economy.
- **Listing Pass prevents spam.** Only creators who invest a Listing Pass can put workflows on the marketplace, maintaining quality.

### Billing Model
Per-execution (pay-as-you-go) + Subscription (heavy users) + One-time purchase (simple automations)

### Example Flow
```
Builder deploys "Invoice Processing Pipeline"
    → purchasePrice: $75
    → executionPrice: $0.15 per invoice processed
    → subscriptionPrice: $30/month unlimited

Buyer (accounting firm) purchases instance
    → Processes 500 invoices/month at $0.15 = $75/month
    → Switches to subscription at $30/month (saves $45/month)
    → Builder earns $30/month recurring
```

---

## 4. Compute & GPU Marketplace

### The Problem
GPU owners and compute providers want to sell spare capacity. Buyers need on-demand compute without long-term cloud commitments. Current decentralized compute networks (Akash, Render) require their own tokens and ecosystems.

### How the Indie Protocol Solves It

- **Each compute offering is a ServiceNFT.** "4x A100 GPU, 1 hour" or "CPU batch processing, per-job" --- each is a tokenized service.
- **Per-execution billing** maps naturally to compute jobs. Each job draws from the TBA based on actual resource consumption.
- **`maxExecutionPrice` prevents bill shock.** A buyer sets a ceiling and the executor halts if a job would exceed it.
- **inUSD eliminates token volatility.** Unlike paying in AKT or RNDR, all billing is in a dollar-pegged stablecoin.

### Billing Model
Per-execution (per-job or per-hour) + Subscription (reserved capacity)

### Example Flow
```
Provider deploys "ML Training - A100 4x"
    → executionPrice: $2.50 per GPU-hour
    → maxExecutionPrice: $100 per job
    → subscriptionPrice: $500/month (reserved allocation)

Buyer (ML startup) purchases instance
    → Funds TBA with $1,000 inUSD
    → Runs 50 training jobs, avg 4 GPU-hours each
    → 200 GPU-hours × $2.50 = $500 deducted from TBA
    → Provider earns $500
```

---

## 5. Data Feed & Oracle Marketplace

### The Problem
Data providers (financial data, weather, blockchain analytics, social sentiment) need a way to sell continuous data feeds with subscription billing. Consumers need to discover and subscribe to reliable feeds with verifiable delivery records.

### How the Indie Protocol Solves It

- **Each data feed is a ServiceNFT** with `subscriptionPrice` as the primary billing model.
- **ERC-5643 subscriptions** handle 30-day recurring billing natively on-chain. No off-chain subscription management needed.
- **BillingExecutor** creates an on-chain record of data delivery (execution count = delivery count), providing verifiable reliability metrics.
- **Auto-cancel on transfer** ensures new owners must explicitly subscribe, preventing unintended billing after secondary market trades.

### Billing Model
Subscription (primary) + Per-execution (on-demand queries)

### Example Flow
```
Provider deploys "Real-Time Crypto Sentiment Feed"
    → subscriptionPrice: $100/month
    → executionPrice: $0.005 per on-demand query

Buyer (hedge fund) purchases instance
    → Subscribes: pays $100 inUSD to provider
    → Receives continuous feed for 30 days
    → Also makes 2,000 on-demand queries → $10 from TBA
    → Renews subscription at day 30
```

---

## 6. SaaS Micro-Service Platform

### The Problem
SaaS founders want to launch micro-services (email validation, PDF generation, translation, OCR) without building billing, auth, and marketplace infrastructure from scratch. Each service is too small for a standalone SaaS but viable as a marketplace offering.

### How the Indie Protocol Solves It

- **Each micro-service is a ServiceNFT.** The protocol provides the complete billing and distribution layer.
- **Three pricing models** let founders experiment. Offer per-call billing for casual users, subscriptions for power users, and one-time purchase for enterprise.
- **Gasless purchase + inUSD** means non-crypto users can onboard via email login and pay in familiar dollar amounts.
- **Feature Pass boosting** gives micro-services visibility without requiring a marketing budget.

### Billing Model
All three models available simultaneously per service

### Example Flow
```
Founder deploys "PDF Invoice Generator"
    → purchasePrice: $10 (lifetime access)
    → executionPrice: $0.02 per PDF
    → subscriptionPrice: $5/month unlimited

Casual user: buys access for $10, pays $0.02 per PDF
Power user: switches to $5/month subscription
Enterprise: negotiates custom terms via direct contract interaction
```

---

## 7. Freelance & Professional Services

### The Problem
Freelancers and consultants (designers, developers, analysts) deliver digital outputs (reports, designs, code audits) with no standard way to tokenize deliverables, prove delivery, and receive instant payment without intermediaries taking 20-30%.

### How the Indie Protocol Solves It

- **Each service offering is a ServiceNFT.** "Brand Identity Design Package" or "Smart Contract Audit" --- each is a tokenized product.
- **One-time purchase model** works naturally for deliverable-based services. Buyer pays `purchasePrice`, receives the instance NFT as proof of purchase.
- **100% revenue to creator.** No Fiverr 20% cut, no Upwork fees. The creator receives the full payment.
- **On-chain purchase record** serves as a verifiable receipt and proof of engagement for both parties.
- **Instance NFT as deliverable receipt.** The NFT can link to the actual deliverable via metadata, creating a permanent, verifiable record.

### Billing Model
One-time purchase (primary) + Per-execution (revisions, follow-up consultations)

### Example Flow
```
Designer deploys "Logo Design Package"
    → purchasePrice: $500
    → executionPrice: $50 per revision round

Client purchases instance → $500 inUSD to designer
    → Receives deliverable linked to NFT metadata
    → Requests 2 revisions → $100 from TBA
    → Designer earns $600 total, 0% commission
```

---

## 8. Education & Content Gating

### The Problem
Course creators, tutorial authors, and educational content producers need fine-grained access control. Current platforms (Udemy, Teachable) take 30-50% of revenue and control the student relationship.

### How the Indie Protocol Solves It

- **Each course or content package is a ServiceNFT.** Access is proven by owning the instance NFT.
- **Subscription model** for ongoing education (monthly access to new content).
- **Per-execution model** for pay-per-lesson or pay-per-assessment.
- **Instance NFT doubles as a credential.** Owning the NFT proves course completion or membership.
- **Transferable access.** Students can resell course access on secondary NFT markets if they no longer need it.

### Billing Model
One-time purchase (course bundles) + Subscription (membership) + Per-execution (individual lessons)

### Example Flow
```
Instructor deploys "Advanced Solidity Security Course"
    → purchasePrice: $200 (lifetime access)
    → subscriptionPrice: $30/month (new content updates)
    → executionPrice: $5 (per-assessment grading)

Student purchases instance → $200 to instructor
    → Accesses all course materials
    → Subscribes for monthly updates → $30/month
    → Takes 4 graded assessments → $20 from TBA
    → Owns NFT as proof of completion
```

---

## 9. IoT & Device Service Marketplace

### The Problem
IoT device manufacturers and service providers offer cloud-connected services (monitoring, analytics, firmware updates, data processing) that need per-device billing. Traditional per-seat SaaS pricing does not map well to device fleets.

### How the Indie Protocol Solves It

- **Each IoT service is a ServiceNFT.** Each device deployment is an instance with its own TBA.
- **Per-execution billing** maps to per-event processing (sensor readings, alerts, data ingestion).
- **TBA per device** isolates billing per deployment. A fleet manager can fund each device's TBA independently.
- **Subscription model** for continuous monitoring (monthly per-device fee).
- **Lock/unlock** maps to active monitoring windows. Locked = device is being monitored. Unlocked = paused.

### Billing Model
Per-execution (event-based) + Subscription (continuous monitoring)

### Example Flow
```
Provider deploys "Industrial Sensor Analytics"
    → executionPrice: $0.001 per sensor reading processed
    → subscriptionPrice: $10/month per device (unlimited readings)

Factory deploys 50 devices, each with an instance
    → Funds each TBA with $20 inUSD
    → 10,000 readings/device/month × $0.001 = $10/device
    → Switches high-volume devices to subscription at $10/month
    → Provider earns $500/month from this customer
```

---

## 10. Decentralized Service-to-Service Economy

### The Problem
In a world of microservices, services need to call and pay other services. Currently this requires bilateral API key exchange, invoice management, and manual reconciliation. There is no trustless service-to-service payment protocol.

### How the Indie Protocol Solves It

- **ERC-8004 agent identity** enables machine-readable service discovery. Service A can find Service B's capabilities programmatically.
- **TBA-to-TBA billing.** Service A's TBA can fund a call to Service B, with Service B's executor drawing the execution fee. No human intervention.
- **BillingExecutor** provides the audit trail for automated inter-service transactions.
- **Composability is native.** Any ServiceNFT can call any other ServiceNFT. The billing protocol is the same regardless of which services interact.

### Billing Model
Per-execution (machine-to-machine calls)

### Example Flow
```
"Content Generator" agent (Service A)
    → Needs image generation
    → Discovers "Image Generator" agent (Service B) via ERC-8004
    → Service A's TBA pays Service B's executionPrice ($0.10)
    → Service B generates image, returns result
    → Service A incorporates image into content
    → End user pays Service A's executionPrice ($0.50)
    → Net revenue: Service A earns $0.40, Service B earns $0.10
```

---

## 11. Tokenized Consulting & Advisory

### The Problem
Advisory services (legal reviews, financial modeling, security audits) are high-value, low-frequency engagements with no standard way to prove delivery, manage payment escrow, or build verifiable reputation across platforms.

### How the Indie Protocol Solves It

- **Each advisory package is a ServiceNFT** with a `purchasePrice` reflecting the engagement value.
- **TBA acts as escrow.** Client funds the TBA; the advisor draws upon deliverable milestones via execution billing.
- **BillingExecutor** records each milestone delivery, creating an on-chain proof of work.
- **Reputation** builds from cumulative execution stats: total engagements completed, total revenue earned, all verifiable on-chain.

### Billing Model
One-time purchase (engagement fee) + Per-execution (milestone-based)

### Example Flow
```
Advisor deploys "Smart Contract Security Audit"
    → purchasePrice: $5,000 (engagement fee)
    → executionPrice: $1,000 per milestone
    → maxExecutionPrice: $2,000 (cap per milestone)

Client purchases instance → $5,000 to advisor
    → Funds TBA with $4,000 for 4 milestones
    → Milestone 1: Initial review → $1,000 from TBA
    → Milestone 2: Detailed findings → $1,000 from TBA
    → Milestone 3: Remediation verification → $1,000 from TBA
    → Milestone 4: Final report → $1,000 from TBA
    → On-chain record: 4 milestones delivered, $9,000 total engagement
```

---

## 12. Gaming & Digital Asset Services

### The Problem
Game developers and digital asset creators offer services (character generation, item crafting, level design, asset transformation) that need metered billing. Current solutions require custom in-game currency infrastructure.

### How the Indie Protocol Solves It

- **inUSD serves as the universal gaming currency** for service payments (no need to mint a custom game token).
- **Per-execution billing** maps to per-action charges (generate a character, craft an item, render a scene).
- **ServiceNFT instances** can represent licenses to use specific generators or tools.
- **Subscription model** for premium tool access (unlimited generations per month).

### Billing Model
Per-execution (per-generation) + Subscription (unlimited access)

### Example Flow
```
Artist deploys "AI Character Portrait Generator"
    → executionPrice: $0.50 per portrait
    → subscriptionPrice: $15/month unlimited

Gamer purchases instance → Funds TBA with $25
    → Generates 30 character portraits → $15 from TBA
    → Switches to subscription → $15/month
    → Artist earns recurring revenue from 500+ subscribers
```

---

## 13. x402 Web Monetization

### The Problem
Content creators and API providers want to monetize individual HTTP requests (articles, API calls, premium content) without requiring user registration or subscription commitment.

### How the Indie Protocol Solves It

- **X402Receiver** enables standard HTTP 402 "Payment Required" responses. The service returns a 402 status with payment details; the client pays via the x402 protocol.
- **No account required.** The buyer pays per-request via USDC through an x402 facilitator. The protocol collects payments and converts to inUSD.
- **Micropayment-native.** Payments as small as $0.001 per request are viable because settlement is batched.

### Billing Model
Per-request (HTTP 402)

### Example Flow
```
Publisher deploys premium article service
    → HTTP GET /article/123 → 402 Payment Required
    → Client pays $0.10 via x402 facilitator
    → X402Receiver collects USDC
    → Admin converts to inUSD → Publisher credited
    → No login, no subscription, no cookie walls
```

---

## Use Case Compatibility Matrix

| Use Case | Purchase | Execution | Subscription | TBA | x402 | ERC-8004 | Passes |
|----------|----------|-----------|-------------|-----|------|----------|--------|
| AI Agents | Yes | Yes | Yes | Yes | - | Yes | Yes |
| API Monetization | Yes | Yes | - | Yes | Yes | - | Yes |
| Workflow Automation | Yes | Yes | Yes | Yes | - | Yes | Yes |
| Compute/GPU | - | Yes | Yes | Yes | - | - | - |
| Data Feeds | - | Yes | Yes | Yes | - | - | - |
| SaaS Micro-Services | Yes | Yes | Yes | Yes | Yes | - | Yes |
| Freelance/Professional | Yes | Yes | - | Yes | - | - | - |
| Education | Yes | Yes | Yes | - | - | - | - |
| IoT Services | - | Yes | Yes | Yes | - | Yes | - |
| Service-to-Service | - | Yes | - | Yes | - | Yes | - |
| Consulting/Advisory | Yes | Yes | - | Yes | - | - | - |
| Gaming/Digital Assets | - | Yes | Yes | Yes | - | - | - |
| x402 Web Monetization | - | - | - | - | Yes | - | - |

---

## Summary: One Protocol, Many Markets

The Indie Protocol is not designed for a single vertical. Its power comes from the **generality of its primitives**:

- **Any service** that can report execution events can use per-execution billing via TBAs
- **Any product** with a fixed price can use the one-time purchase model
- **Any service** with recurring value can use ERC-5643 subscriptions
- **Any machine-readable service** can participate in the service-to-service economy via ERC-8004
- **Any web service** can monetize individual requests via x402

The protocol provides the economic layer. Execution is always external and pluggable. This means every use case above can coexist on the same marketplace, sharing the same payment rails, the same discovery mechanisms, and the same reputation system.
