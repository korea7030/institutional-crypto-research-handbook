---
knowledge_id: ONCHAIN-METRICS-013
title: TVL
subtitle: Total Value Locked as a Capital-Allocation Metric, Not a Universal Health Score
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - DeFi
parent:
  - Onchain Metrics
prerequisites:
  - DEFI-012
related_topics:
  - Stablecoin Supply
  - Metric Limitations
primary_sources:
  - REF-LLAMA-TVL-2026-001
tags:
  - tvl
  - defi
---

# TVL
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-013

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-013
title: TVL
research_question: >
  What does total value locked actually aggregate, and why can TVL overstate
  protocol health when capital is recursive, subsidized, or price-driven?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-012
parent: Onchain Metrics
previous: ONCHAIN-METRICS-012
next: ONCHAIN-METRICS-014
```

## 1. Learning Objectives

- Define TVL.
- Explain how TVL rises through both net inflows and asset-price appreciation.
- Identify recursion and double-counting risk.

## 2. Executive Summary

DefiLlama tracks TVL as the aggregate value held across DeFi protocols and chains.[^ref-llama-tvl]

TVL is useful for understanding capital allocation and protocol relevance, but it is often misused as a universal quality metric.

## 3. Why TVL Can Mislead

- price appreciation can inflate TVL without new capital,
- recursive collateral use can double count,
- incentive programs can attract mercenary liquidity,
- bridge-wrapped assets can echo the same economic capital in multiple layers.

## 4. Institutional Thinking

- TVL is a stock-of-capital metric, not a profitability metric.
- Pair TVL with fees, users, retention, and risk concentration.

## 5. References

[^ref-llama-tvl]: DefiLlama methodology and protocol TVL dashboards, accessed 2026-08-04, https://defillama.com

## 6. Cross References

- Previous: ONCHAIN-METRICS-012 — Stablecoin Supply
- Next: ONCHAIN-METRICS-014 — Network Growth

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
