---
knowledge_id: ONCHAIN-METRICS-001
title: Exchange Flows
subtitle: Deposit and Withdrawal Tracking as a Lens on Market Intent
version: 1.0.0
status: Reviewed
difficulty: L300
estimated_reading: 85 min
estimated_study: 200 min
last_reviewed: 2026-08-04
domain:
  - Onchain Metrics
  - Market Structure
parent:
  - Onchain Metrics
prerequisites:
  - DEFI-003
related_topics:
  - Exchange Reserve
  - Whale Activity
  - Metric Limitations
primary_sources:
  - REF-CM-FLOWIN-2026-001
  - REF-CM-FLOWOUT-2026-001
tags:
  - exchange-flows
  - onchain-metrics
---

# Exchange Flows
> Onchain Metrics  
> Research Unit: ONCHAIN-METRICS-001

---

## Research Brief

```yaml
knowledge_id: ONCHAIN-METRICS-001
title: Exchange Flows
research_question: >
  What do exchange inflow and outflow metrics measure, how are they constructed
  from labeled exchange entities, and what can and cannot be inferred from flow
  spikes?
document_type: foundation
difficulty: L300
prerequisites:
  - DEFI-003
parent: Onchain Metrics
previous:
next: ONCHAIN-METRICS-002
```

## 1. Learning Objectives

- Define exchange inflow and outflow.
- Explain dependence on exchange address labeling.
- Distinguish flow observation from trader motive.
- Use flow data as contextual evidence rather than single-factor proof.

## 2. Executive Summary

Coin Metrics documents exchange inflow and outflow style metrics as transfer activity involving exchange-labeled entities.[^ref-cm-flowin][^ref-cm-flowout]

Analysts often interpret large inflows as potential sell pressure and large outflows as potential self-custody or reduced immediate sell availability. That interpretation can be useful, but it is not guaranteed. Internal treasury shuffling, omnibus-wallet management, and cross-venue routing can all distort surface-level conclusions.

## 3. Core Mechanics

Exchange flow metrics depend on:

- identifying which addresses belong to exchanges,
- aggregating transfers into and out of those entities,
- deciding whether internal entity movements are excluded or consolidated.

Because the metric rests on entity labeling, methodology quality matters as much as raw transfer volume.

## 4. Analytical Use

Exchange inflow spikes are often watched around:

- panic conditions,
- post-rally profit taking,
- derivative collateral movements.

Exchange outflow spikes are often watched around:

- long-term custody shifts,
- ETF or custodian withdrawals,
- rotation into DeFi or staking.

## 5. Metric Limits

- Not all exchange wallets are known.
- Some known exchange flows are internal.
- A deposit to an exchange does not prove immediate sale.
- A withdrawal does not prove accumulation.

## 6. Institutional Thinking

- Treat exchange flows as intent-adjacent evidence, not direct proof of action.
- Require context from price, derivatives, reserve, and entity-concentration data.

## 7. References

[^ref-cm-flowin]: Coin Metrics Docs, exchange inflow metric documentation and transfer methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

[^ref-cm-flowout]: Coin Metrics Docs, exchange outflow metric documentation and transfer methodology, accessed 2026-08-04, https://docs.coinmetrics.io/network-data/network-data-overview

## 8. Cross References

- Next: ONCHAIN-METRICS-002 — Exchange Reserve

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
