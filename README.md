Solana Rug Guard (Entity-Level Risk Intelligence)




Solana Rug Guard is an on-chain risk intelligence system focused on early-stage token risk detection and entity-level wallet attribution.  
Rather than relying on social signals or price movement, it builds an evolving entity graph and evaluates structural patterns behind token launches.

This repository is a public-facing system overview for enterprise deployment and integration discussions.
<img width="1536" height="1024" alt="1" src="https://github.com/user-attachments/assets/0ec8c16a-4c75-4987-b86a-d38cad78d117" />


 Why Solana

Solana’s high-throughput execution and rich transaction metadata enable real-time attribution across deployment wallets, funders, and liquidity operators.  
That makes entity-level risk intelligence feasible at launch-time and at first-trade, not hours later.

 Architecture (Single Pipeline)

```text
                 ┌───────────────────────────────────────────┐
                       (Webhook Listener ) → Event Parser    │
                 └──────────────────────┬────────────────────┘
                                        │ mint / lp / swap
                                        v
                              ┌───────────────────┐
                              │  Single Pipeline   │
                              │  - store events    │
                              │  - build features  │
                              └───────┬───────────┘
                                      │
              ┌───────────────────────┼────────────────────────┐
              │                       │                        │
              v                       v                        v
     ┌────────────────┐     ┌────────────────────┐   ┌──────────────────┐
     │ Prewarn (Mint) │     │ Entity Resolver     │   │ Final (FirstSwap)│
     │ - no score     │     │ - authority/funding │   │ - score + grade   │
     │ - entity hints │     │ - entity graph      │   │ - reputation fuse │
     └───────┬────────┘     └──────────┬─────────┘   └─────────┬────────┘
              \_________________________│_______________________/
                                        v
                           ┌────────────────────────────┐
                           │ Outputs                    │
                           │ - Webhook / API / Telegram │
                           │ - JSONL outbox             │
                           └────────────────────────────┘
```

Why This Exists

Early token risk is rarely just “a token problem”. It is usually an operator/entity problem:

 The same team reuses authorities, fee payers, funders, liquidity operators, and deployment patterns
 Risk is inherited across launches
 The most valuable signal is not the chart — it is the control and funding structure

Solana Rug Guard is designed to surface those structures early and consistently.

Core Principles (Expert Priority)

Entity clustering and attribution follow a strict evidence priority:

1. **Authority Chain (Highest Priority)**  
   Mint Authority / Freeze Authority / Update Authority represent absolute control.
2. **Funding Chain (Strongest Entity Evidence)**  
   Fee payer and initial funder relationships are strong indicators of common operators.
3. **Trading Behavior (Reinforcement Evidence)**  
   Early clustering / self-trade patterns are treated as supporting signals.
4. **LP Behavior (Post-event Evidence)**  
   Strong but often happens later; used as high-weight reinforcement when available.

This ordering is designed from a project-operator risk perspective, not a chart-first perspective.

 System Overview

High-level flow:

1. Real-time event intake via webhooks/streams
2. Normalization & filtering to keep only launch-relevant traces
3. Feature extraction (authority, funding, early trading, liquidity structure)
4. Entity resolution (wallet roles clustered into behavioral entities)
5. Signal emission (two-stage)
6. Outputs (Webhook/API/Telegram/JSON outbox)

 Two-Stage Signals (Non-Misleading)

To balance speed and accuracy, signals are emitted in two stages:

 1 Prewarn (Mint-Time Intelligence)

Triggered when a new mint is detected.

- Outputs: entity attribution (if resolvable), authority structure, funding hints
- Does not output a final score/grade
- Goal: early intelligence without premature conclusions

 2 Final (First-Swap Scoring)

Triggered after the first on-chain swap (or equivalent activation condition).

 Outputs: risk score, grade, evidence summary, entity reputation impact, links
 Goal: actionable scoring once sufficient data exists

This reduces noise from “empty mints” (no LP / no swaps) and avoids premature judgment.

 Entity Intelligence (V1–V4 Ready)

 V1 — Entity Identification (Core)

Clusters wallets into an entity using strong evidence:

 Authorities (mint/freeze/update)
 Fee payer
 Initial funder (funding trace)

Outputs per token:
 `entity_id`
 entity history counters (risk occurrences)

 V2 — Entity Profile (Historical Reputation)

Each entity accumulates a historical profile over time:

 number of launches
 high-risk/rug frequency
 authority-not-renounced frequency
 LP withdrawability frequency
 early cluster frequency

 V3 — Reputation Fusion (Scoring Integration)

Entity reputation becomes a scoring dimension:

 new tokens inherit risk weight from the entity’s history
 outputs include entity history summary and tags

 V4 — Pattern Library & Entity Graph

Beyond single-entity attribution, relationship links are built from patterns such as:

 shared fee payer across projects
 shared funder across projects
 inter-entity transfer hints
 LP structure reuse patterns (including fine-grained pool params)

Outputs:
 `entity_links` and related-entity hints

 What We Detect (High-Level, No Rule Leakage)

The system evaluates multiple structural dimensions, including:

 control and permissions structure (authorities, token program traits)
 funding origin and upstream funding traces
 liquidity provisioning patterns and LP safety indicators
 early distribution anomalies and trading behavior clusters
 cross-token entity reuse and relationship graph patterns

This repository intentionally does not publish thresholds, weights, or proprietary heuristics.

 Deployment (Enterprise-Friendly)

Runtime components:

 Webhook listener (ingest)
 Single pipeline processor (parse → filter → store → resolve entity → emit signals)
 API service (optional)
 JSON outbox (optional) for downstream ingestion

Typical API capabilities:

 latest signals: `GET /signals`
 entity list: `GET /entities`
 entity details: `GET /entity?id=...`
 lookup by address: `GET /entity-address?address=...`
 lookup by mint: `GET /entity-mint?mint=...`

 Quick Start / Integration

For enterprise integration we support:

 outbound webhooks (recommended)
 API pull (optional)

If you share a webhook URL and preferred auth header, we can deliver stable JSON payloads for `prewarn` and `final` signals.

 Roadmap

 V1–V2: entity identification + historical profile enrichment
 V3: reputation fusion into scoring
 V4: pattern library + entity network graph
 V5: commercial risk intelligence layer (subscription APIs, graph explorer) — out of scope for this public overview

 Disclaimer

This system provides risk intelligence and structural attribution signals.  
It does not claim certainty of fraud, and should be used as part of a broader risk management workflow.  
It is not financial advice and does not guarantee detection of all malicious activity.

 Contact / Enterprise Integration

If you are an exchange, wallet provider, market maker, or security team, reach out for integration details.

Contact: gauss8008@gmail.com
X: @LeviathanVcz
