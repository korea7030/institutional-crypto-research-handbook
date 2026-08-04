---
knowledge_id: ONCHAIN-METRICS-009
title: Dormancy
subtitle: Coin-Age Consumption as a Window into Holder Patience and Spend Intensity
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
  - ONCHAIN-METRICS-008
related_topics:
  - Coin Days Destroyed
  - SOPR
primary_sources:
  - REF-GN-DORM-2026-001
tags:
  - dormancy
  - holder-behavior
---

# Dormancy
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-009

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-009
title: Dormancy
research_question: >
  What does coin dormancy measure, how is it related to coin-age spending, and
  why can spikes suggest older holder distribution without proving capitulation
  or tops by themselves?
document_type: foundation
difficulty: L400
prerequisites:
  - ONCHAIN-METRICS-008
parent: Onchain Metrics
previous: ONCHAIN-METRICS-008
next: ONCHAIN-METRICS-010
```

## 1. Learning Objectives

- Define dormancy.
- Distinguish dormancy from transaction count.
- Explain why old-coin movement matters.

## 2. Executive Summary

Glassnode documents Average Coin Dormancy as the average number of days spent outputs remained dormant before being moved.[^ref-gn-dorm]

Dormancy therefore asks not just how many coins moved, but how old they were when they moved.

## 3. Interpretation

High dormancy can suggest:

- old holders distributing,
- long-inactive supply reawakening,
- structural reallocation by early cohorts.

Low dormancy can suggest dominance of younger coin movement.

## 4. Limits

- exchange internal reorganizations can pollute data,
- old-coin movement does not always mean discretionary selling,
- cohort context matters.

## 5. Institutional Thinking

- Dormancy is most informative when segmented by cohort and paired with realized-profit metrics.

## 6. References

[^ref-gn-dorm]: Glassnode Docs, "Dormancy" metric guide, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides/dormancy

## 7. Cross References

- Previous: ONCHAIN-METRICS-008 — NUPL
- Next: ONCHAIN-METRICS-010 — Coin Days Destroyed

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
