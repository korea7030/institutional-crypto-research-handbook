---
knowledge_id: ONCHAIN-METRICS-006
title: SOPR
subtitle: Spent Output Profit Ratio as a Realized Profit-and-Loss Lens
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 95 min
estimated_study: 220 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Valuation
parent:
  - Onchain Metrics
prerequisites:
  - BITCOIN-014
related_topics:
  - MVRV
  - NUPL
primary_sources:
  - REF-GN-SOPR-2026-001
tags:
  - sopr
  - pnl
---

# SOPR
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-006

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-006
title: SOPR
research_question: >
  What does SOPR measure about realized profit and loss on spent coins, and why
  is the 1.0 threshold often treated as a behavioral regime marker?
document_type: foundation
difficulty: L400
prerequisites:
  - BITCOIN-014
parent: Onchain Metrics
previous: ONCHAIN-METRICS-005
next: ONCHAIN-METRICS-007
```

## 1. Learning Objectives

- Define SOPR.
- Explain why SOPR above or below 1 matters.
- Distinguish realized from unrealized profitability.

## 2. Executive Summary

Glassnode defines SOPR as the ratio between the realized value in USD and the value at creation of spent outputs, effectively capturing whether spent coins are moving at profit or loss.[^ref-gn-sopr]

If SOPR is above `1`, spent coins are realizing aggregate profit. If below `1`, spent coins are realizing aggregate loss.[^ref-gn-sopr]

## 3. Formula

Conceptually:

`SOPR = Realized Value / Value at Creation`

across spent outputs in the interval.[^ref-gn-sopr]

## 4. Interpretation

Analysts often watch `1.0` as a psychological and structural threshold:

- above `1`: holders can realize gains,
- below `1`: realized loss-taking dominates.

## 5. Limits

- SOPR only observes spent outputs.
- It says nothing directly about coins not moving.
- Exchange and consolidation activity can contaminate short windows.

## 6. Institutional Thinking

- SOPR is strongest when combined with trend, holder segmentation, and macro positioning.

## 7. References

[^ref-gn-sopr]: Glassnode Docs, "SOPR" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/sopr

## 8. Cross References

- Previous: ONCHAIN-METRICS-005 — Active Addresses
- Next: ONCHAIN-METRICS-007 — MVRV

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
