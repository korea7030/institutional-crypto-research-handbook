---
knowledge_id: ONCHAIN-METRICS-011
title: Realized Cap
subtitle: Cost-Basis-Oriented Valuation from Last Onchain Movement
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 90 min
estimated_study: 210 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Valuation
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-010
related_topics:
  - MVRV
  - NUPL
primary_sources:
  - REF-GN-REALCAP-2026-001
tags:
  - realized-cap
  - valuation
---

# Realized Cap
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-011

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-011
title: Realized Cap
research_question: >
  What is realized capitalization, why does it differ from market cap, and how
  does it function as an aggregate cost-basis proxy for onchain valuation work?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-010
parent: Onchain Metrics
previous: ONCHAIN-METRICS-010
next: ONCHAIN-METRICS-012
```

## 1. Learning Objectives

- Define realized cap.
- Distinguish realized cap from market cap.
- Explain why realized cap underpins multiple onchain valuation metrics.

## 2. Executive Summary

Glassnode documents realized capitalization as valuing each coin at the price when it last moved onchain rather than at current spot price.[^ref-gn-realcap]

This reframes the network not as today's mark-to-market sum, but as an aggregate cost-basis estimate embedded in UTXO history.

## 3. Why It Matters

Realized cap is a base layer for:

- MVRV,
- NUPL,
- cost-basis style valuation work,
- market-cycle stress analysis.

## 4. Limits

- offchain transfer of ownership is invisible,
- wrapped and custodial structures can blur economic ownership,
- last onchain move is not always true acquisition cost.

## 5. Institutional Thinking

- Realized cap is one of the most useful bridges between blockchain data and behavioral valuation, but it remains a proxy.

## 6. References

[^ref-gn-realcap]: Glassnode Docs, "Realized Cap" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/realized-cap

## 7. Cross References

- Previous: ONCHAIN-METRICS-010 — Coin Days Destroyed
- Next: ONCHAIN-METRICS-012 — Stablecoin Supply

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
