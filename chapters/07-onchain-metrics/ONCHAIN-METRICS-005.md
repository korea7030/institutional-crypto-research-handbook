---
knowledge_id: ONCHAIN-METRICS-005
title: Active Addresses
subtitle: Participation Proxy, Usage Signal, and the Need to Separate Activity from Users
version: 1.0.0
status: Reviewed
difficulty: L200
estimated_reading: 80 min
estimated_study: 180 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Network Activity
parent:
  - Onchain Metrics
prerequisites:
  - ONCHAIN-METRICS-004
related_topics:
  - Network Growth
  - Metric Limitations
primary_sources:
  - REF-CM-ACTIVEADDR-2026-001
tags:
  - active-addresses
  - network-activity
---

# Active Addresses
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-005

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-005
title: Active Addresses
research_question: >
  What do active-address metrics actually count, and why must analysts avoid
  equating address activity with unique human-user growth?
document_type: foundation
difficulty: L200
prerequisites:
  - ONCHAIN-METRICS-004
parent: Onchain Metrics
previous: ONCHAIN-METRICS-004
next: ONCHAIN-METRICS-006
```

## 1. Learning Objectives

- Define active addresses.
- Distinguish activity count from user count.
- Explain why batching and contract usage distort the metric.

## 2. Executive Summary

Coin Metrics documents active-address metrics as counts of addresses that were active in transactions during a period.[^ref-cm-active]

This is a useful participation proxy. It is not a direct measure of unique users. One user can control many addresses, and one address can represent exchange batching or application-level aggregation.

## 3. Interpretation

Rising active addresses may indicate:

- stronger transactional activity,
- renewed speculation,
- growing application use,
- or even spam and bot behavior.

## 4. Limits

- exchange batching can suppress apparent counts,
- UTXO and account-based systems behave differently,
- contract architectures change how many addresses appear active.

## 5. Institutional Thinking

- Use active addresses as a network-activity clue, not a direct adoption count.
- Pair with transaction counts, fees, and cohort context.

## 6. References

[^ref-cm-active]: Coin Metrics Docs, active-address metric definitions such as `AdrActCnt`, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 7. Cross References

- Previous: ONCHAIN-METRICS-004 — Smart Money
- Next: ONCHAIN-METRICS-006 — SOPR

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
