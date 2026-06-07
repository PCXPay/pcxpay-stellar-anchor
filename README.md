# PCX Anchor — Regulated Stellar Anchor for Emerging Market Corridors

> Built by PCXPAY | Circle Alliance Programme Member | 
> Triple Licensed: FCA · CBN IMTO · FINTRAC

PCXPAY is building the first regulated Stellar anchor 
connecting GBP, Nigerian Naira, Euro, and Canadian Dollars 
to USDC on the Stellar network — enabling instant, low-cost 
cross-border payments for fintechs and businesses serving 
African-corridor markets.

---

## The Problem

Legacy payment infrastructure was not built for the 
transaction shapes modern fintechs need across emerging 
markets:

- **Micropayments** below €1 are uneconomical on SWIFT 
  due to fixed correspondent banking fees of $15–30 per 
  transaction
- **Settlement** takes days, not seconds
- **Bi-directional flows** — moving money both in and out 
  of Africa through the same infrastructure — are 
  unsupported by most providers

Our founding experiment: we attempted a €0.50 cross-border 
payout and found no existing rail could process it 
economically. That gap is the product.

---

## What We Are Building

A rail-agnostic payments middleware platform where Stellar 
serves as the optimal settlement layer for African and 
emerging market corridors. Fintechs integrate our single 
API and get access to multiple payment rails — traditional 
banking and USDC on Stellar — without needing to understand 
the underlying infrastructure.

---

## Stellar Integration Stack

| Component | SEP / Tool | Purpose | Status |
|---|---|---|---|
| Hosted deposit/withdrawal | SEP-24 | Fiat on/off-ramp via hosted UI | Building |
| Cross-border anchor payments | SEP-31 | Direct B2B anchor-to-anchor payments | Building |
| FX quoting / rate lock | SEP-38 | Real-time quote before transaction | Building |
| KYC data exchange | SEP-12 | Compliance data between anchor and client | Building |
| Human-readable addresses | SEP-2 Federation | user*PCXPAY.com address resolution | Building |
| Fiat ↔ USDC conversion | Bridge | Manages all fiat/USDC conversion | Integrating |
| Cash off-ramps | MoneyGram Ramps | Cash pickup in 180+ countries | Integrating |
| Bulk payouts / payroll | Stellar Disbursement Platform | B2B enterprise bulk payments | Integrating |
| Smart contracts / escrow | Soroban | Future phase — programmable payments | Planned |

---

## Architecture Overview

PCXPAY's existing infrastructure is a serverless AWS 
Lambda stack with multiple payment partners, rate 
aggregation, and a compliance layer. The Stellar 
integration adds the following components:

### Orchestration Layer

**Routing Engine**
Evaluates cost, speed, and reliability per transaction and 
selects the optimal rail. Stellar is selected dynamically 
when it is the cheapest and fastest option for a given 
corridor. Traditional rails remain available as fallback.

**Tenant Config Service**
Per-client rate margins, corridor permissions, compliance 
rules, and feature flags for multi-tenant B2B2C support.

**FX Rate Aggregator**
Consolidates rates from multiple providers. Feeds SEP-38 
RFQ responses. Cached in ElastiCache/Redis for performance.

### Stellar Rail Components

**SDF Anchor Platform** — deployed on ECS Fargate (not 
Lambda). Handles SEP-24, SEP-31, SEP-12, SEP-38. Publishes 
stellar.toml. Registered for GBP, NGN, EUR, CAD.

**Federation Server** — human-readable addresses: 
user*PCXPAY.com via SEP-2.

**Bridge (USDC Manager)** — manages all fiat↔USDC 
conversion and liquidity. PCXPAY does not hold or 
rebalance USDC directly.

**MoneyGram Ramps** — cash pickup in 180+ countries via 
Stellar-exclusive API. Enables unbanked recipients to 
collect payments in local cash.

**Stellar Disbursement Platform** — bulk payouts in hosted 
mode for B2B payroll product.

**Stellar Horizon Monitor** — network health, transaction 
confirmations, and anchor status observability.

### Core Services Extended

**Internal Ledger** — double-entry bookkeeping on Aurora 
Serverless. ACID-compliant, per-tenant isolation. Replaces 
DynamoDB for financial records.

**Travel Rule Engine** — FATF Travel Rule compliance for 
cross-border Stellar transactions above regulatory 
thresholds.

**OFAC Screening** — sanctions screening on all parties 
and all transactions before processing.

---

## Payment Flows

### B2B2C — Fintech's user sends money

Fintech calls Platform API → Routing Engine selects Stellar → Anchor Platform handles SEP-31 → Bridge converts fiat to USDC → USDC settles on Stellar (~2.5 seconds) → Receiving anchor converts to local fiat → Payout via local rails → Webhook fires to fintech

### B2B — Bulk payroll

Business uploads CSV → SDP processes batch → Bridge converts bulk amount to USDC → SDP distributes to recipients on Stellar → Local anchors handle fiat payout → Recipients without bank accounts collect via MoneyGram cash pickup

### Internal transfer — both users on PCXPAY

Federation Server resolves recipient address → Internal Ledger credits and debits atomically → Instant settlement, zero cost, zero network dependency

---

## Build Phases and Milestones

| Phase | Deliverable | Target Date |
|---|---|---|
| 1 | Anchor Platform on testnet. stellar.toml published. Federation server live. SEP-24 + SEP-12 on testnet for GBP and NGN. Routing Engine prototype. | 31 Aug 2026 |
| 2 | SEP-31 + SEP-38 on mainnet — GBP→NGN corridor live via USDC. Full payment state machine. MoneyGram Ramps initiated. Additional corridors (EUR, CAD) on testnet. | 31 Oct 2026 |
| 3 | Full production launch. All corridors live on mainnet. SDP bulk payouts. MoneyGram cash off-ramp active. Developer Portal with sandbox. Horizon Monitor. | 31 Jan 2027 |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Compute | AWS Lambda (core services) + ECS Fargate (Anchor Platform) |
| Database | DynamoDB (operational) + Aurora Serverless v2 (Internal Ledger) |
| Cache | ElastiCache / Redis (FX rates, idempotency, sessions) |
| Queue | SQS + Dead Letter Queue (payment retries, event processing) |
| Stellar | SDF Anchor Platform, Stellar Disbursement Platform, Horizon |
| Settlement | USDC on Stellar via Bridge |
| Cash access | MoneyGram Ramps |
| KYC/KYB | Bridge KYC + Bridge KYB |
| Compliance | Travel Rule Engine, OFAC Screening |
| Infrastructure | Terraform / AWS CDK |

---

## Why Stellar Over Alternatives

| Capability | Lightning Network | Stellar (PCXPAY) |
|---|---|---|
| Settlement asset | Bitcoin (volatile) | USDC (stable, 1:1 USD) |
| Liquidity management | Complex channel balancing | None required (Bridge) |
| Cash off-ramps | Not available | MoneyGram (180+ countries) |
| Bulk disbursements | Not native | SDP (open-source) |
| Smart contracts | Limited | Soroban (Rust, full capability) |
| Settlement speed | ~seconds | 2.5 seconds (no routing failures) |
| Multi-currency support | Not native | Path payments (atomic) |

---

## Traction

- Live with paying clients processing real cross-border volume
- Lead client processing ~£500K/month
- 100+ payments completed across four emerging markets
- Pipeline of 4 enterprise fintech clients at demo stage
- Government tender submitted: £2.31M
- Circle Alliance Programme member: 
  https://partners.circle.com/partner/PCXPAY
- Triple licensed: FCA · CBN IMTO · FINTRAC
- SCF #44 Build Award interest form approved: May 2026

---

## Developer Resources

- [Stellar Anchor Platform](https://developers.stellar.org/docs/platforms/anchor-platform)
- [SEP-24 Guide](https://developers.stellar.org/docs/platforms/anchor-platform/sep-guide/sep24)
- [SEP-31 Cross-Border Guide](https://developers.stellar.org/docs/platforms/anchor-platform/sep-guide/sep31)
- [SEP-38 RFQ Guide](https://developers.stellar.org/docs/platforms/anchor-platform/sep-guide/sep38)
- [MoneyGram Ramps](https://developer.moneygram.com/moneygram-developer/docs/integrate-moneygram-ramps)
- [Stellar Disbursement Platform](https://github.com/stellar/stellar-disbursement-platform-backend)
- [Bridge API](https://bridge.xyz/docs)
- [SCF Handbook](https://stellar.gitbook.io/scf-handbook)

---

## Contact

- Website: https://pcxpay.com
- Circle Alliance: https://partners.circle.com/partner/PCXPAY
- Email: info@PCXPAY.com
