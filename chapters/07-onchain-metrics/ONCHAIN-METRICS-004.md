---
knowledge_id: ONCHAIN-METRICS-004
title: Smart Money
subtitle: Labeled Sophisticated Wallets, Signal Extraction, and Survivorship Bias
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Entity Analysis
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-003
related_topics:
  - Whale Activity
  - Metric Limitations
primary_sources:
  - REF-NANSEN-SMARTMONEY-2026-001
tags:
  - smart-money
  - wallet-labels
---

# Smart Money
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-004

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-004
title: Smart Money
research_question: >
  What do labeled smart-money metrics try to capture, how are such labels built,
  and what biases emerge when analysts infer quality from historical wallet
  performance?
document_type: foundation
difficulty: L300
prerequisites:
  - ONCHAIN-METRICS-003
parent: Onchain Metrics
previous: ONCHAIN-METRICS-003
next: ONCHAIN-METRICS-005
```

## 1. Learning Objectives

- Define smart-money metrics as label-based heuristics.
- Explain why wallet-quality labels are model outputs, not consensus facts.
- Identify survivorship and lookback bias.

## 2. Executive Summary

Nansen documents smart-money style wallet labeling as part of an entity-classification framework that attempts to identify sophisticated and historically successful participants.[^ref-nansen-smart]

This can be useful because raw addresses are otherwise anonymous. But smart-money dashboards are only as good as:

- the label set,
- the performance criteria,
- the lookback window,
- and the exclusion logic.

## 3. Why It Appeals

Analysts want to know where apparently informed capital is rotating before the narrative becomes obvious.

## 4. Biases

- survivorship bias,
- hindsight classification,
- label leakage,
- strategy-regime changes.

## 5. Institutional Thinking

- Smart-money labels are research shortcuts, not ground truth.
- Use them as lead generation, then verify with direct wallet and protocol context.

## 6. References

[^ref-nansen-smart]: Nansen Docs, smart money and wallet-label methodology overview, accessed 2026-08-04, https://docs.nansen.ai

## 7. Cross References

- Previous: ONCHAIN-METRICS-003 — Whale Activity
- Next: ONCHAIN-METRICS-005 — Active Addresses

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
