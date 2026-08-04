---
knowledge_id: ONCHAIN-METRICS-015
title: Metric Limitations
subtitle: Label Error, Structural Drift, and Why Onchain Data Needs Model Discipline
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 90 min
estimated_study: 210 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Research Method
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-014
related_topics:
  - Smart Money
  - TVL
  - Exchange Flows
primary_sources:
  - REF-CM-NETDATA-2026-001
  - REF-GN-METGUIDES-2026-001
  - REF-ETH-BRIDGES-2026-001
tags:
  - limitations
  - methodology
---

# Metric Limitations
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-015

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-015
title: Metric Limitations
research_question: >
  What are the major failure modes in onchain metrics, and how should analysts
  separate raw blockchain observability from the modeling assumptions required to
  turn data into investment evidence?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-014
parent: Onchain Metrics
previous: ONCHAIN-METRICS-014
next:
```

## 1. Learning Objectives

- Identify the main limitation classes in onchain analytics.
- Distinguish observable base data from modeled entity inference.
- Build a checklist for metric skepticism.

## 2. Executive Summary

Onchain metrics feel objective because blockchain data is public. But many market-relevant metrics require heavy modeling on top of raw chain data.

Coin Metrics network-data documentation shows how metrics depend on definitions, transformations, and entity treatment.[^ref-cm-netdata]

Glassnode metric guides similarly demonstrate that widely cited signals often embed construction choices and heuristic thresholds.[^ref-gn-guides]

Ethereum's bridge documentation highlights another structural problem: economic activity now spans wrapped assets and multiple chains, which weakens naive single-chain interpretation.[^ref-eth-bridges]

## 3. Limitation Classes

- entity-label error,
- address != user,
- internal transfer contamination,
- cross-chain fragmentation,
- recursion and double counting,
- structural breaks from wallet or protocol design changes,
- overfit historical thresholds.

## 4. Research Discipline

Before trusting a metric, ask:

1. What is directly observed?
2. What is inferred?
3. What labels are required?
4. What changed structurally since prior cycles?
5. What alternative explanations fit the same print?

## 5. Institutional Thinking

- Metrics are evidence inputs, not conclusions.
- Strong research comes from triangulation, not metric absolutism.

## 6. References

[^ref-cm-netdata]: Coin Metrics Docs, network-data overview and metric methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

[^ref-gn-guides]: Glassnode Docs, metric guides and indicator methodologies, accessed 2026-08-04, https://docs.glassnode.com/guides-and-tutorials/metric-guides

[^ref-eth-bridges]: ethereum.org, "Bridges," official documentation, page last update 2026-04-03, accessed 2026-08-04, https://ethereum.org/developers/docs/bridges

## 7. Cross References

- Previous: ONCHAIN-METRICS-014 — Network Growth
- Phase 7 complete

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
