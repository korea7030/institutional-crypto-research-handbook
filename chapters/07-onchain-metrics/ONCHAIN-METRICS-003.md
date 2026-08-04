---
knowledge_id: ONCHAIN-METRICS-003
title: Whale Activity
subtitle: Large-Holder Behavior, Concentration Clues, and the Limits of Address-Based Inference
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Ownership
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-002
related_topics:
  - Smart Money
  - Exchange Flows
primary_sources:
  - REF-CM-ADDRBAL-2026-001
tags:
  - whales
  - concentration
---

# Whale Activity
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-003

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-003
title: Whale Activity
research_question: >
  How should analysts study whale activity using address-balance and transfer
  data, and why does address concentration only imperfectly map to beneficial
  ownership?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-002
parent: Onchain Metrics
previous: ONCHAIN-METRICS-002
next: ONCHAIN-METRICS-004
```

## 1. Learning Objectives

- Define whale activity without relying on vague social-media usage.
- Explain the difference between large addresses and large beneficial owners.
- Use concentration and transfer size metrics cautiously.

## 2. Executive Summary

Coin Metrics documents balance-distribution metrics such as counts of addresses above specific balance thresholds.[^ref-cm-addrbal]

These metrics are often used as proxies for whale activity. They can be useful for identifying concentration shifts, accumulation by large cohorts, or large transfer regimes. But one address can represent many users, and one user can control many addresses.

## 3. Common Proxies

- number of addresses above threshold,
- supply held by top balance bands,
- large-transfer counts,
- exchange inflow and outflow by large entities.

## 4. Interpretive Cautions

- exchange wallets can look like whales,
- custody fragmentation can hide whales,
- threshold choices can arbitrarily change the story.

## 5. Institutional Thinking

- Whale analysis is best done at the cohort and entity level where possible.
- Avoid presenting address-level concentration as direct proof of beneficial-owner concentration.

## 6. References

[^ref-cm-addrbal]: Coin Metrics Docs, balance-distribution and address-count metric documentation, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 7. Cross References

- Previous: ONCHAIN-METRICS-002 — Exchange Reserve
- Next: ONCHAIN-METRICS-004 — Smart Money

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
