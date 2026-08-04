---
knowledge_id: ONCHAIN-METRICS-002
title: Exchange Reserve
subtitle: Exchange-Held Supply as a Supply Availability Signal
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Market Structure
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-001
related_topics:
  - Exchange Flows
  - Stablecoin Supply
primary_sources:
  - REF-CM-EXCHSUPPLY-2026-001
tags:
  - exchange-reserve
  - supply
---

# Exchange Reserve
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-002

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-002
title: Exchange Reserve
research_question: >
  What does exchange reserve measure, how is it related to labeled exchange-held
  balances, and why should analysts separate supply availability from certain
  directional price forecasting?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-001
parent: Onchain Metrics
previous: ONCHAIN-METRICS-001
next: ONCHAIN-METRICS-003
```

## 1. Learning Objectives

- Define exchange reserve.
- Explain why reserve differs from flow.
- Explain how exchange reserve can matter for supply-overhang narratives.
- Identify reserve-labeling limitations.

## 2. Executive Summary

Exchange reserve metrics estimate the amount of asset supply held by exchange entities at a point in time.[^ref-cm-exchsupply]

Where exchange flows focus on movement over an interval, exchange reserve focuses on stock. Analysts often use falling reserve as evidence of tightening immediately available supply and rising reserve as evidence of growing on-venue inventory. That framing is useful, but it remains conditional on labeling quality and venue structure.

## 3. Core Mechanics

Reserve is conceptually:

`Exchange Reserve = Sum of balances across labeled exchange-controlled addresses`

The hard problem is not the arithmetic. It is the entity map.

## 4. Interpretation

Potentially informative patterns:

- declining reserve during long accumulation phases,
- rising reserve before heavy distribution,
- diverging reserve behavior across BTC and stablecoins.

## 5. Limitations

- omnibus wallets can obscure end-user ownership,
- custodians and exchanges can overlap,
- cross-chain wrappers complicate net reserve understanding,
- off-exchange OTC inventory is invisible.

## 6. Institutional Thinking

- Reserve is a supply-location metric, not a direct sell-order metric.
- Always pair reserve with flows, price, and derivatives positioning.

## 7. References

[^ref-cm-exchsupply]: Coin Metrics Docs, exchange supply and entity-balance methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 8. Cross References

- Previous: ONCHAIN-METRICS-001 — Exchange Flows
- Next: ONCHAIN-METRICS-003 — Whale Activity

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
