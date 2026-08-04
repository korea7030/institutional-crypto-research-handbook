---
knowledge_id: ONCHAIN-METRICS-012
title: Stablecoin Supply
subtitle: Dry Powder Narratives, Settlement Liquidity, and the Need to Separate Gross Issuance from Deployable Demand
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Liquidity
parent:
  - Onchain Metrics
prerequisites:
  - TOKEN-STANDARDS-004
related_topics:
  - Exchange Reserve
  - TVL
primary_sources:
  - REF-GN-SSR-2026-001
  - REF-LLAMA-STABLES-2026-001
tags:
  - stablecoin-supply
  - liquidity
---

# Stablecoin Supply
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-012

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-012
title: Stablecoin Supply
research_question: >
  How should analysts use stablecoin supply as a liquidity and risk-appetite
  signal, and why are gross stablecoin balances not automatically equivalent to
  deployable demand for crypto assets?
document_type: foundation
difficulty: L300
prerequisites:
  - TOKEN-STANDARDS-004
parent: Onchain Metrics
previous: ONCHAIN-METRICS-011
next: ONCHAIN-METRICS-013
```

## 1. Learning Objectives

- Define stablecoin-supply metrics.
- Explain dry-powder narratives and their limits.
- Distinguish issuance from deployable risk appetite.

## 2. Executive Summary

Glassnode documents the Stablecoin Supply Ratio (SSR) as a relationship between Bitcoin market cap and aggregate stablecoin supply, using stablecoin issuance as a liquidity proxy.[^ref-gn-ssr]

DefiLlama also maintains stablecoin supply tracking across chains and issuers as a market-structure dataset.[^ref-llama-stables]

Stablecoin supply matters because it can indicate:

- settlement liquidity,
- fiat onboarding intensity,
- risk appetite,
- and collateral availability.

## 3. Limits

- stablecoins may sit idle,
- supply may be fragmented across chains,
- issuer actions can change supply without immediate market deployment.

## 4. Institutional Thinking

- Stablecoin supply is a liquidity backdrop metric, not direct proof of imminent spot buying.

## 5. References

[^ref-gn-ssr]: Glassnode Docs, "Stablecoin Supply Ratio (SSR)" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/stablecoin-supply-ratio-ssr

[^ref-llama-stables]: DefiLlama Docs and dashboards, stablecoin methodology and chain/issuer supply tracking, accessed 2026-08-04, https://defillama.com/stablecoins

## 6. Cross References

- Previous: ONCHAIN-METRICS-011 — Realized Cap
- Next: ONCHAIN-METRICS-013 — TVL

## Review Status

### Technical Review
Passed.

### Evidence Review
Passed.

### Editorial Review
Passed.

### Adversarial Review
Passed.

### Quality Gate

| Gate | Status |
|---|---|
| Research scope was followed | Pass |
| Required primary sources were reviewed | Pass |
| Material claims are cited | Pass |
| Fact and interpretation are separated | Pass |
| No unresolved critical review issue remains | Pass |
