---
knowledge_id: ONCHAIN-METRICS-008
title: NUPL
subtitle: Net Unrealized Profit and Loss as Aggregate Embedded PnL
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
  - ONCHAIN-METRICS-007
related_topics:
  - MVRV
  - SOPR
primary_sources:
  - REF-GN-NUPL-2026-001
tags:
  - nupl
  - unrealized-pnl
---

# NUPL
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-008

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-008
title: NUPL
research_question: >
  What does NUPL estimate about aggregate unrealized profit and loss embedded in
  the network, and how should researchers use it without turning heuristic
  sentiment bands into deterministic signals?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-007
parent: Onchain Metrics
previous: ONCHAIN-METRICS-007
next: ONCHAIN-METRICS-009
```

## 1. Learning Objectives

- Define NUPL.
- Explain its relation to realized and market capitalization.
- Use sentiment bands cautiously.

## 2. Executive Summary

Glassnode defines NUPL as the difference between relative unrealized profit and relative unrealized loss, commonly simplified to the normalized gap between market cap and realized cap.[^ref-gn-nupl]

Conceptually:

`NUPL = (Market Cap - Realized Cap) / Market Cap`[^ref-gn-nupl]

## 3. Interpretation

- positive NUPL: unrealized profit dominates,
- negative NUPL: unrealized loss dominates.

Analysts often map NUPL into cycle bands, but those bands are descriptive heuristics rather than fixed laws.

## 4. Limits

- embedded cost basis can lag structural changes,
- sentiment labels can overfit past cycles,
- regime persistence can be longer than expected.

## 5. Institutional Thinking

- Use NUPL as a regime thermometer, not a one-day signal generator.

## 6. References

[^ref-gn-nupl]: Glassnode Docs, "NUPL" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/nupl

## 7. Cross References

- Previous: ONCHAIN-METRICS-007 — MVRV
- Next: ONCHAIN-METRICS-009 — Dormancy

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
