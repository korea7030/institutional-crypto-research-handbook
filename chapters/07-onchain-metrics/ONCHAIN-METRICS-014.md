---
knowledge_id: ONCHAIN-METRICS-014
title: Network Growth
subtitle: New Address Formation, Usage Expansion, and Adoption Inference
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Network Activity
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-005
related_topics:
  - Active Addresses
  - Metric Limitations
primary_sources:
  - REF-CM-NEWADDR-2026-001
tags:
  - network-growth
  - adoption
---

# Network Growth
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-014

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-014
title: Network Growth
research_question: >
  How should analysts use new-address and first-use style metrics as adoption
  evidence, and why are network-growth signals vulnerable to wallet-architecture
  changes and artificial activity?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-005
parent: Onchain Metrics
previous: ONCHAIN-METRICS-013
next: ONCHAIN-METRICS-015
```

## 1. Learning Objectives

- Define network-growth style metrics.
- Distinguish new addresses from new users.
- Explain why address creation policy matters.

## 2. Executive Summary

Coin Metrics documents address-creation and first-funding style metrics such as new funded address count.[^ref-cm-newaddr]

These metrics can be useful growth proxies because they attempt to capture fresh address participation rather than only repeated activity from existing addresses. But wallet architecture, exchange batching, spam, and protocol design can all change these counts without true adoption shifts.

## 3. Institutional Thinking

- Network growth should be read with active addresses, fees, retained balances, and application-level usage.
- Fast address growth with weak retention can be less meaningful than slower but sticky user expansion.

## 4. References

[^ref-cm-newaddr]: Coin Metrics Docs, new-address and first-funded-address metric documentation, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 5. Cross References

- Previous: ONCHAIN-METRICS-013 — TVL
- Next: ONCHAIN-METRICS-015 — Metric Limitations

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
