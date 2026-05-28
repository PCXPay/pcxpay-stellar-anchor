# PCX Anchor — Regulated Stellar Anchor for Emerging Market Corridors

PCX Pay is building a regulated Stellar anchor connecting GBP, Nigerian Naira, Euro, and Canadian Dollars to USDC on the Stellar network — enabling instant, low-cost cross-border payments for fintechs and businesses serving African-corridor markets.

## What We're Building

A rail-agnostic payments middleware platform where Stellar serves as the optimal settlement layer for African and emerging market corridors. Fintechs integrate our single API and get access to multiple payment rails — traditional banking and USDC on Stellar — without ever needing to understand the underlying infrastructure.

## Stellar Integration Stack

| Component | Implementation | Status |
|---|---|---|
| Hosted deposit/withdrawal | SEP-24 | Build |
| Cross-border anchor payments | SEP-31 | Build |
| FX quoting / rate lock | SEP-38 | Build |
| KYC data exchange | SEP-12 | Build |
| Human-readable addresses | SEP-2 Federation | Build |
| Fiat ↔ USDC conversion | Bridge | Ready |
| Cash off-ramps (180+ countries) | MoneyGram Ramps | Integrate |
| Bulk payouts / payroll | Stellar Disbursement Platform | Integrate |
| Smart contracts / escrow | Soroban | Phase 2 |

## Architecture Overview

PCX Pay's existing infrastructure is a serverless AWS Lambda stack with multiple payment partners, rate aggregation, and a compliance layer. The Stellar integration adds the following new components:

**Orchestration Layer**
- Routing Engine — evaluates cost, speed, and reliability per transaction and selects the optimal rail. Stellar is selected dynamically when it is the cheapest and fastest option for a given corridor.
- Tenant Config Service — per-client rate margins, corridor permissions, compliance rules, and feature flags for multi-tenant B2B2C support.
- FX Rate Aggregator — consolidates rates from six providers, feeds SEP-38 RFQ responses, cached in ElastiCache/Redis.

**Stellar Rail**
- SDF Anchor Platform — deployed on ECS Fargate (not Lambda). Handles SEP-24, SEP-31, SEP-12, SEP-38. Publishes stellar.toml. Registered for GBP, NGN, EUR, CAD.
- Federation Server — human-readable addresses: user*pcxpay.com via SEP-2.
- Bridge (USDC Manager) — manages all fiat↔USDC conversion and USDC liquidity. PCX Pay does not hold or rebalance USDC directly.
- MoneyGram Ramps — cash pickup in 180+ countries via Stellar-exclusive API.
- Stellar Disbursement Platform — bulk payouts in hosted mode for B2B payroll product.
- Stellar Horizon Monitor — network health, transaction confirmations, anchor status observability.

**Core Services (Extended)**
- Internal Ledger — double-entry bookkeeping on Aurora Serverless. ACID-compliant, per-tenant isolation. Replaces DynamoDB for financial records.
- Virtual Accounts — extended to support Stellar path payments for atomic multi-hop currency conversion.
- Travel Rule Engine — FATF Travel Rule compliance for cross-border Stellar transactions.
- OFAC Screening — sanctions screening on all parties and all transactions.

## Payment Flows

**B2B2C (Fintech's user sends money)**
Fintech calls Platform API → Routing Engine selects Stellar → Anchor Platform handles SEP-31 → Bridge converts fiat to USDC → USDC settles on Stellar → receiving anchor converts to local fiat → payout via local rails → webhook fires to fintech.

**B2B (Bulk payroll)**
Business uploads CSV → SDP processes batch → Bridge converts bulk amount to USDC → SDP distributes to recipients → local anchors handle fiat payout → recipients without apps collect via MoneyGram cash pickup.

**Internal transfer (both users on PCX Pay)**
Federation Server resolves recipient address → Internal Ledger credits and debits atomically via Aurora → instant settlement, zero cost, zero network dependency.

## Build Phases & SCF Milestones

| Phase | Deliverable | SCF Tranche |
|---|---|---|
| 1 | Anchor Platform on testnet. stellar.toml published. Federation server live. | T0: $15K |
| 2 | SEP-24 + SEP-12 on testnet. Routing Engine prototype. ElastiCache deployed. | T1: $30K |
| 3a | SEP-31 + SEP-38 on mainnet — GBP→NGN corridor live via USDC. Full payment state machine. | T2: $45K |
| 3b | MoneyGram Ramps. Additional corridors (EUR, CAD). Production multi-tenant Platform API. | T2 |
| 4 | Full production launch. SDP bulk payouts. Developer Portal with sandbox. Horizon Monitor. | T3: $60K |

**Total SCF request: $150,000 in XLM**

## Tech Stack

- **Compute:** AWS Lambda (core services) + ECS Fargate (Anchor Platform)
- **Database:** DynamoDB (operational data) + Aurora Serverless v2 (Internal Ledger)
- **Cache:** ElastiCache / Redis (FX rates, idempotency keys, sessions)
- **Queue:** SQS + Dead Letter Queue (payment retries, event processing)
- **Stellar:** SDF Anchor Platform, Stellar Disbursement Platform, Stellar Horizon
- **Settlement:** USDC on Stellar via Bridge
- **Cash access:** MoneyGram Ramps
- **KYC/KYB:** Bridge KYC + Bridge KYB (PCX KYC/KYB service planned)
- **Compliance:** Travel Rule Engine, OFAC Screening
- **IaC:** Terraform / AWS CDK

## Why Stellar

| Capability | Lightspark (alternative) | Stellar (PCX Pay) |
|---|---|---|
| Settlement asset | Bitcoin (volatile) | USDC (stable, 1:1 USD) |
| Liquidity management | Complex channel balancing | None required (Bridge manages) |
| Cash off-ramps | Not available | MoneyGram (180+ countries) |
| Bulk disbursements | Not native | SDP (open-source, purpose-built) |
| Smart contracts | Limited | Soroban (Rust, full capability) |
| Multi-currency wallets | Not native | Path payments (atomic conversion) |
| Settlement speed (Q4 2026) | ~seconds | 2.5 seconds |

## Traction

- Live with paying clients processing real cross-border transaction volume
- Lead client processing ~£500K/month
- 100+ payments completed across four emerging markets
- Pipeline of 4 enterprise fintech clients at demo stage
- Circle Alliance Programme member: https://partners.circle.com/partner/pcxpay
- Triple licensed across three jurisdictions

## Developer Resources

- [Stellar Anchor Platform](https://developers.stellar.org/docs/platforms/anchor-platform)
- [SEP-31 Cross-Border Guide](https://developers.stellar.org/docs/platforms/anchor-platform/sep-guide/sep31)
- [MoneyGram Ramps](https://developer.moneygram.com/moneygram-developer/docs/integrate-moneygram-ramps)
- [Stellar Disbursement Platform](https://github.com/stellar/stellar-disbursement-platform-backend)
- [Bridge API](https://bridge.xyz/docs)
- [SCF Handbook](https://stellar.gitbook.io/scf-handbook)

---

**Contact:** info@pcxpay.com | https://pcxpay.com | https://partners.circle.com/partner/pcxpay
