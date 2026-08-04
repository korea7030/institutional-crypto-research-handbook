---
knowledge_id: ONCHAIN-METRICS-007
title: MVRV
subtitle: Market Value vs Realized Value as a Relative Valuation Framework
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
  - ONCHAIN-METRICS-006
related_topics:
  - NUPL
  - Realized Cap
primary_sources:
  - REF-GN-MVRV-2026-001
tags:
  - mvrv
  - valuation
---

# MVRV
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-007

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-007
title: MVRV
research_question: >
  How does MVRV compare market capitalization with realized capitalization, and
  why is it commonly used as a cycle-aware valuation signal rather than a
  short-term trading trigger?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-006
parent: Onchain Metrics
previous: ONCHAIN-METRICS-006
next: ONCHAIN-METRICS-008
```

## 1. Learning Objectives

- Define MVRV.
- Explain why realized cap is used as cost basis proxy.
- Use MVRV as regime context, not single-point certainty.

## 2. Executive Summary

Glassnode documents MVRV as the ratio of market cap to realized cap.[^ref-gn-mvrv]

The intuition is that market cap reflects current spot valuation, while realized cap approximates the aggregate value at which coins last moved. Their ratio therefore gives a rough measure of how far current valuation has stretched relative to embedded cost basis.

## 3. Formula

`MVRV = Market Cap / Realized Cap`[^ref-gn-mvrv]

## 4. Interpretation

- high MVRV: large unrealized profit embedded in the network,
- low MVRV: compressed valuation relative to realized cost basis.

## 5. Limits

- no universal threshold works across all regimes,
- supply migration patterns and market structure evolve,
- derivatives and ETF channels can weaken direct comparability across cycles.

## 6. Institutional Thinking

- MVRV is more useful for valuation regime framing than precise timing.

## 7. References

[^ref-gn-mvrv]: Glassnode Docs, "MVRV Ratio" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/mvrv

## 8. Cross References

- Previous: ONCHAIN-METRICS-006 — SOPR
- Next: ONCHAIN-METRICS-008 — NUPL

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
