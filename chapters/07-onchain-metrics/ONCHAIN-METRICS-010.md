---
knowledge_id: ONCHAIN-METRICS-010
title: Coin Days Destroyed
subtitle: Age-Weighted Transfer Activity and Supply Reactivation
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 90 min
estimated_study: 210 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Holder Behavior
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-009
related_topics:
  - Dormancy
  - SOPR
primary_sources:
  - REF-GN-CDD-2026-001
tags:
  - cdd
  - coin-days-destroyed
---

# Coin Days Destroyed
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-010

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-010
title: Coin Days Destroyed
research_question: >
  How does Coin Days Destroyed weight transaction activity by coin age, and why
  can it reveal supply reactivation more effectively than raw transfer counts?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-009
parent: Onchain Metrics
previous: ONCHAIN-METRICS-009
next: ONCHAIN-METRICS-011
```

## 1. Learning Objectives

- Define Coin Days Destroyed.
- Explain age weighting.
- Distinguish CDD from raw onchain volume.

## 2. Executive Summary

Glassnode documents Coin Days Destroyed as the destruction of accumulated coin days when previously dormant coins move.[^ref-gn-cdd]

If one coin sits idle for 100 days and then moves, it destroys more coin days than one coin that moved yesterday.

## 3. Formula

Conceptually:

`CDD = Sum(Coin Amount * Days Since Last Move)`[^ref-gn-cdd]

## 4. Interpretation

- high CDD: older supply is active,
- low CDD: transfer activity is dominated by younger supply.

## 5. Institutional Thinking

- CDD is especially useful when price rises on unusually old-coin activity, suggesting historically patient holders are participating.

## 6. References

[^ref-gn-cdd]: Glassnode Docs, "CDD (Coin Days Destroyed)" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/cdd

## 7. Cross References

- Previous: ONCHAIN-METRICS-009 — Dormancy
- Next: ONCHAIN-METRICS-011 — Realized Cap

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
