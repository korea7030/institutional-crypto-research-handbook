---
knowledge_id: DEFI-011
title: MEV
subtitle: Transaction Ordering Value, Searcher Competition, and the Hidden Market Inside Block Production
version: 1.0.0
status: Reviewed
difficulty: L400
estimated_reading: 110 min
estimated_study: 270 min
last_reviewed: 2026-08-04
domain:
  - DeFi
  - Market Structure
  - Ethereum
parent:
  - DeFi
prerequisites:
  - DEFI-003
  - DEFI-006
related_topics:
  - Liquidation
  - Bridges
  - DeFi Risks
primary_sources:
  - REF-ETH-MEV-2026-001
tags:
  - mev
  - searchers
  - validators
---

# MEV
> DeFi  
> Research Unit: DEFI-011

---

## Research Brief

```yaml
knowledge_id: DEFI-011
title: MEV
research_question: >
  What is maximal extractable value, how is it created by transaction ordering
  control, and why does MEV reshape execution quality, validator incentives, and
  DeFi protocol design?
document_type: foundation
difficulty: L400
prerequisites:
  - DEFI-003
  - DEFI-006
parent: DeFi
previous: DEFI-010
next: DEFI-012
```

## 1. Learning Objectives

- Define MEV precisely.
- Explain searchers and validators in the MEV supply chain.
- Connect MEV to arbitrage and liquidation.
- Explain centralization implications.

## 2. Executive Summary

Ethereum.org defines maximal extractable value as the maximum value extractable from block production beyond standard block rewards and gas fees by including, excluding, and reordering transactions in a block.[^ref-eth-mev]

The same page explains that, in practice, searchers identify opportunities and pay high fees for inclusion, while validators capture part of the value by controlling block production.[^ref-eth-mev]

## 3. Sources of MEV

- DEX arbitrage,
- liquidations,
- sandwiching and other order-flow exploitation,
- backruns around state changes.

## 4. Economic Structure

If searcher opportunity value is `V_mev` and fee paid for inclusion is `F`, rational searchers compete until:

`F <= V_mev`

and in competitive cases `F` may approach most of the available value.[^ref-eth-mev]

## 5. Security and Centralization

Ethereum.org notes that MEV can accelerate validator centralization because it materially affects proposer economics under PoS.[^ref-eth-mev]

## 6. Institutional Thinking

- MEV is not an edge phenomenon; it is embedded in transparent programmable markets.
- Many DeFi user-experience problems are really transaction-ordering problems.

## 7. References

[^ref-eth-mev]: ethereum.org, "Maximal extractable value (MEV)," official documentation, accessed 2026-08-04, https://ethereum.org/developers/docs/mev

## 8. Cross References

- Previous: DEFI-010 — Bridges
- Next: DEFI-012 — DeFi Risks

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
